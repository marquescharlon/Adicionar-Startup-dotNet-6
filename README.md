# Classe Startup.cs no .NET Core 6
Ela nunca foi obrigatória, mas surgiu para organizar o nosso projeto. Porém, a partir da versão do .NET Core 6 por padrão ela já não é mais criada. No entanto, isso não impede de você implementá-la. Isso poderá lhe ser útil caso precise migrar um projeto antigo para a nova versão do .NET Core.

[marquescharlon/Full-Stack-Developer](https://github.com/marquescharlon/Full-Stack-Developer)

## 1. Criar a classe Startup.cs

Na pasta raiz do seu projeto crie um novo arquivo do tipo **Class** seguindo o caminho no Visual Studio: <br>
`Add > New item > Startup.cs`

> A título de curiosidade, a estrutura básica da Startup.cs é:

```
namespace myApp
{
    public class Startup
    {
        public Startup(IConfiguration configuration) { }
        public IConfiguration Configuration { get; }
        
        public void ConfigureServices(IServiceCollection services) { }
        public void Configure(WebApplication app, IWebHostEnvironment environment) { }
     }
}
```

## 2. Adaptar a classe Startup.cs
Só a estrutura básica da classe Startup.cs não irá funcionar. Então, precisamos extrair da classe Program.cs o conteúdo que virá para nosso novo arquivo e ficará da seguinte forma:

```
namespace myApp
{
    public class Startup : IStartup
    {
        public Startup(IConfiguration configuration) { }
        public IConfiguration Configuration { get; }
        
        // Aqui você configura os serviços, seus middlewares
        public void ConfigureServices(IServiceCollection services)
        {
            services.AddControllers();
            services.AddEndpointsApiExplorer();
            services.AddSwaggerGen();
        }
        
        // Aqui você diz que quer usar seus middlewares
        public void Configure(WebApplication app, IWebHostEnvironment environment)
        {
            if (app.Environment.IsDevelopment())
            {
                app.UseSwagger();
                app.UseSwaggerUI();
            }
            app.UseHttpsRedirection();
            app.UseAuthorization();
            app.MapControllers();
        }
    }

    public interface IStartup
    {
        IConfiguration Configuration { get; }
        void Configure(WebApplication app, IWebHostEnvironment environment);
        void ConfigureServices(IServiceCollection services);
    }
    
    public static class StartupExtensions
    {
        public static WebApplicationBuilder UseStartup<TStartup>(this WebApplicationBuilder WebAppBuilder) where TStartup : IStartup
        {
            var startup = Activator.CreateInstance(typeof(TStartup), WebAppBuilder.Configuration) as IStartup;
            if (startup == null) throw new ArgumentException("Classe Startup.cs inválida!");

            startup.ConfigureServices(WebAppBuilder.Services);

            var app = WebAppBuilder.Build();
            startup.Configure(app, app.Environment);

            app.Run();

            return WebAppBuilder;
        }
    }
}
```

## 3. Classe Program.cs
Agora, nesse arquivo iremos apagar tudo e apenas chamar a Startup.cs da seguinte forma:

```
using myApp;

var builder = WebApplication.CreateBuilder(args)
    .UseStartup<Startup>();
```

# Conclusão
Com apenas essas alterações sua aplicação funcionará normalmente, agora, contendo sua classe **Startup.cs** facilitando a migração de uma aplicação antiga para o **.NET Core 6**.
