## O uso de threads do BackgroundService no .NET Core  

Recentemente precisei escrever um algoritmo onde uma classe herdada de `BackgroundService` - classe abstrata que implementa `IHostedService`, a interface que roda fluxos em segundo plano - rodava infinitamente. A regra de negócio era executada apenas 1 vez no mês, mas mesmo assim você precisa de uma thread para verificar se essa _é a vez do mês_.

`

class MyCustomWorker : BackgroundService
{
    private const int RunAt = 1;

    protected override async Task ExecuteAsync(CancellationToken stoppingToken)
    {
        while (!stoppingToken.IsCancellationRequested)
        {
            if (DateTime.UtcNow.Day == RunAt && DateTime.UtcNow.Hour == RunAt)
            {
                await DoAsyncWork();
            }
        }
    }
}

`

Quando esse worker é injetado com `AddHostedService<T>`, qualquer outro worker injetado também com `AddHostedService<T>` não será mais executado - e isso se deve pela forma em que o runtime do .NET gerencia a inicialização desses workers.

Entender o motivo desse problema te força a entender um pouco mais sobre assincronicidade e sincronicidade, um conceito muito mais complexo do que se parece a primeira instância.

## Threads

_Na ciência da computação, a thread é a menor sequência de instruções programadas que podem ser gerenciadas independentemente por um scheduler, que normalmente faz parte do sistema operacional._

![alt text](/images/2025-05-15-background-services-dotnet/thread.png)
_Fonte: Wikipedia_

Ou seja, dentro de cada thread há um fluxo de código que você define. Tudo o que roda por um programa de computador é executado por pelo menos uma thread (e várias delas podem ser executadas ao mesmo tempo).

Quando se é usado um `await Async()`, a thread é liberada para o thread-pool se `Async()` for I/O bound (ou seja, um processo que não precise de uma thread, como por exemplo esperar o resultado de um banco de dados ou de uma requisição HTTP).

Isso significa que, enquanto um fluxo que não está utilizando nenhuma thread é realizado (e eu preciso esperar o resultado dele), eu devolvo a thread que eu estava usando até então para o thread-pool - e lá algum processo pode utilizar essa thread para um processo CPU bound (ou seja, um processo que de fato precise de uma thread da CPU).

## Inicialização do BackgroundService pelo runtime do .NET Core

 `await AAsync();
 await BAsync();
 await CAsync();`

 Ao usar o `await`, o código é executado de forma sequencial, ou seja, `BAsync()` só será executado quando `AAsync()` for finalizado e `CAsync()` só será executado quando `BAsync()` for finalizado. O ganho, nesse caso, é você liberar a thread para o thread-pool para outros processos utilizá-lo enquanto você espera pela resposta de cada método.

 Dito isso,

 `AAsync();
BAsync();
CAsync();`

 não usar o `await` executará `AAsync()`, `BAsync()` e `CAsync()` de forma assíncrona, mesmo que na definição desses métodos hajam operações `await`.

Mas `AAsync()` pode travar para sempre mesmo sem um `await` em sua chamada, e é exatamente isso que acontece com o `BackgroundService`.

Na [definição do construtor do `StartyAsync()` do runtime do .NET](https://github.com/dotnet/runtime/blob/e3ffd343ad5bd3a999cb9515f59e6e7a777b2c34/src/libraries/Microsoft.Extensions.Hosting.Abstractions/src/BackgroundService.cs#L37), o método `ExecuteAsync()` - que é o nosso worker - é chamado sem um `await`.

Como resultado, o runtime irá iniciar a execução do worker e, sem esperar pelo resultado dele, retornará um `Task.CompletedTask`, finalizando assim a execução de `StartAsync()`. Isso faz sentido, pois a ideia não é o runtime esperar pelo término de cada worker (afinal é um trabalho em segundo plano), mas sim apenas inicializá-los.

Agora, importante entender como o `StartAsync()` é chamado pelo runtime.

Na [documentação da Microsoft](https://learn.microsoft.com/en-us/aspnet/core/fundamentals/host/hosted-services?view=aspnetcore-9.0&tabs=visual-studio#:~:text=StartAsync%20should%20be%20limited%20to%20short%20running%20tasks%20because%20hosted%20services%20are%20run%20sequentially%2C%20and%20no%20further%20services%20are%20started%20until%20StartAsync%20runs%20to%20completion.), é especificado que o `StartAsync()` é chamado de forma sequencial, ou seja, com o uso do `await`. Simplificando, o runtime faz o equivalente de:

`await myCustomWorker.StartAsync();
await smsWorker.StartAsync(); 
await cleanRecordsWorker.StartAsync();`

Dado a definição de `StartAsync()`, sabemos que o runtime vai apenas iniciar cada worker e, sem esperar por seu resultado, irá nos retornar o `Task.CompletedTask` - fazendo assim com que o `await myCustomWorker.StartAsync()` seja finalizado, depois `await smsWorker.StartAsync()` e depois `await cleanRecordsWorker.StartAsync()`, tudo de forma basicamente instantânea.

No entanto, se nossos workers não liberarem a thread instantaneamente (e a thread é liberada ao fazer o uso do `await`, como explicado), a linha

`// Store the task we're executing
_executingTask = ExecuteAsync(_stoppingCts.Token);`

do construtor de `StartAsync()` ficará travada esperando a liberação da thread. Ela está "esperando", mas sem o `await`. A thread ainda não foi liberada para a continuação do fluxo do construtor (e como consequência os outros workers não são inicializados).

Na definição dada de `MyCustomWorker`, nenhum `await` é executado - logo, nenhuma thread é liberada e consequentemente a chamada de `ExecuteAsync()` irá travar. Com a chamada de `ExecuteAsync()` travada, o runtime não vai finalizar a execução de `await myCustomWorker.StartAsync()` (porque estamos esperando o término com o `await`) e nenhum outro worker será executado.

Esse design faz sentido pois se nenhum `await` é utilizado (e o `await` é utilizado para operações I/O bound), uma operação CPU bound está sendo executada - e como sabemos, uma operação CPU bound precisa de... threads de CPU.

Na prática, se o worker inicia com uma lógica síncrona que demora 10 segundos e só depois um `await` é executado, a chamada de `ExecuteAsync()` no construtor de `StartAsync()` só será liberada depois dos 10 segundos.

`class MyCustomWorker : BackgroundService
{
    private const int RunAt = 1;

    protected override async Task ExecuteAsync(CancellationToken stoppingToken)
    {
        MetodoCPUBoundQueDemora10Segundos(); // aqui o construtor tá travado (pois estamos usando threads)
        var dados = await BuscaDadosNoBancoAsync(); // aqui a thread é liberada e o construtor continua seu fluxo

        while (!stoppingToken.IsCancellationRequested)
        {
            if (DateTime.UtcNow.Day == RunAt && DateTime.UtcNow.Hour == RunAt)
            {
                await DoAsyncWork();
            }
        }
    }
}`

Um loop, como é o caso do `MyCustomWorker` na linha `while (!stoppingToken.IsCancellationRequested)`, também é uma operação CPU bound, ou seja, um worker que inicia com um looping também irá travar a chamada de `ExecuteAsync()` - mas dessa vez irá travar pra sempre!

## A solução

Se uma operação CPU bound é executada no início do nosso worker, não queremos que o runtime espere o término dessa operação. Ele pode continuar o seu fluxo e inicializar os outros workers e, depois, essa operação CPU bound arranja uma outra thread no thread-pool. Pra liberar a thread sem necessariamente `await`ar algo, basta usar `await Task.Yield()`:

`class MyCustomWorker : BackgroundService
{
    private const int RunAt = 1;

    protected override async Task ExecuteAsync(CancellationToken stoppingToken)
    {
        await Task.Yield();

        while (!stoppingToken.IsCancellationRequested)
        {
            if (DateTime.UtcNow.Day == RunAt && DateTime.UtcNow.Hour == RunAt)
            {
                await DoAsyncWork();
            }
        }
    }
}`

`await Task.Delay(1)` também funciona pelo fato de liberarmos a thread no `await`. `Task.Yield()` é a forma formal de se fazer isso.

Com isso, qualquer código lento ou qualquer loop não irá travar a execução de `ExecuteAsync()` e todos os outros workers serão inicializados corretamente.