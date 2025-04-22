# M09-T1

Arnau Pascual

## Principis de Programació segura

### Principi de menor privilegi

Cada component, procés o usuari només ha de tenir els permisos mínims necessaris per fer la seva tasca.

### Validació i sanejament d'entrada

Tota entrada s'ha de validar i netejar per evitar injeccions, errors o abusos.

### Principi de la mínima exposició

No mostrar ni retornar més informació de la necessària.

### Principi d'autenticació i autoritzaciós segura

Verificar correctament la identitat i assegurar que cada acció només la pugui fer qui estigui autoritzat.

### Anàlisis, proves i auditories de seguretat

**Aquest no és un privilegi, però si una pràctica de seguretat recomanada.**

Revisar i provar el codi per detectar vulnerablitats.

## Exercicis

### 1. Quina és la diferència entre un socket i un websocket? Quin dels dos pot ser més segur? Quines implementacions poden fer-lo més segur?

Un socket és una connexió entre dos sistemes, una interfície bidireccional entre el client i el servidor mitjançant els protocols TCP i UDP.

Un websocket és una connexió bidireccional i persistents entre el client i el servidor amb una única connexió TCP, aquesta està pensada per a navgeadors i entorns web.

Els websockets són més segurs, ja que per defecte utilitzen https.

Per a fer els websockets més segurs, es pot autenticar l'usuari abans d'establir la connexió a més de posar l'imits de temps a la connexió.

### 2. Descriu els passos a realitzar amb SignalR un sistema per veure els valors de la borsa en temps real. No s’ha d’implementar.

1. Instal·lar llibreries als projectes client i servidor de SignalR.
1. Definir la classe Hub
1. Configurar al servidor amb els mètodes d'enviament de missatges.
1. Configurar al client i implementar amb js la rebuda dels missatges.

### 3. Un diari local vol publicar les seves notícies al seu web. Les notícies es publiquen per categories. Un usuari del web pot veure els titulars i la data de les notícies. Si vol llegir l’article sencer (titular, subtítol, data i hora, redacció, autor i imatge de portada) ha d'autenticar-se com a subscriptor. Els periodistes, prèviament identificats, redacten les notícies i les inserten a la base de dades a través d’un formulari. Una notícia no es publicarà fins que un editor la revisa i la valida. L’editor pot modificar l’article.

**Desenvolupes una APIrest que gestiona les notícies i els usuaris.**
- **Implementa les entitats necessàries i les dades de cada una.**
- **Descriu els rols d’usuari i descriu la política de seguretat de privilegis que té cada rol.**
- **Defineix la classe del controlador que gestiona les notícies i els mètodes. Assegurat que compleix els requisits d’autorització necessaris. Defineix de manera òptima els endpoints i els mètodes HTTP. Fes servir els DTO’s que consideris.**
- **Escriu com hauria de ser el missatge JSON del body del response per afegir una nova notícia.**
- **Descriu els passos per afegir els rols fent servir la llibreria ASPNET.IDENTITY.ENTITYFRAMEWORK.**

#### Entitats

```
public class Article
{
	public int Id { get; set; }
	public string Title { get; set; }
	public string Subtitle { get; set; }
	public DateTime Date { get; set; }
	public string Text { get; set; }
	public string Author { get; set; }
	public string Image { get; set; }
}
```

```
public class ArticleListDTO
{
	public int Id { get; set; }
	public string Title { get; set; }
	public DateTime Date { get; set; }
}
```

```
public class ArticleDTO
{
	public string Title { get; set; }
	public string Subtitle { get; set; }
	public DateTime Date { get; set; }
	public string Text { get; set; }
	public string Author { get; set; }
	public string Image { get; set; }
}
```

#### Rols

Tothom: No és un rol real que s'hagui de crear, però té permisos. Pot veure els titulars i dates de tots els articles

Subscriptor: Pot llegir amb detall tots els articles

Periodista: Pot insertar articles, però no es mostren fins que un Editor ho comprovi

Editor: Pot modificar articles i validar els articles dels Periodistes

#### Controlador

```
[Route("api/[controller]")]
[ApiController]
public class ArticlesController : ControllerBase
{
	// GET: Return ArticleListDTO
	[HttpGet]
	public async Task GetArticles()

	// GET: Return ArticleDTO
	[Authorize]
	[HttpGet("{id}")]
	public async Task GetArticle(int id)

	[Authorize(Roles = "Periodista,Editor")]
	[HttpPut("{id}")]
	public async Task PutArticle(int id, ArticleDTO article)

	[Authorize(Roles = "Periodista,Editor")]
	[HttpPost]
	public async Task PostArticle(ArticleDTO article)

	[Authorize(Roles = "Editor")]
	[HttpDelete]
	public async Task PostArticle(int id)
}
```

#### Json Response

```
{
	"Title" : "Hola!",
	"Subtitle" : "Bon dia",
	"Date" : "2015-12-08T15:15:19",
	"Text" : "AAAAAAAAAAAAAAAAAAAAAAAAAAAA",
	"Author" : "Yo",
	"Image" : "./Images/img.png"
}
```

#### Afegir els rols

1. Afegir la classe IdentityRole al DbContext en la herencia de IdentityDbContext.

```
public class AppDbContext : IdentityDbContext<AppUser, IdentityRole>
{ 
    //…
}
```

2. Configurar el Program.cs, afegir IdentityRole i carregar els rols inicials

```
builder.Services.AddIdentity<ApplicationUser, IdentityRole>()
    .AddEntityFrameworkStores<ApplicationDbContext>()
    .AddDefaultTokenProviders();

//…

var app = builder.Build();

//…

//Carreguem els rols inicials
using (var scope = app.Services.CreateScope())
{
    var services = scope.ServiceProvider;
    await CrearRolsInicials(services);
}

//…

//Afegim el middleware d’autoritzacions i d’autenticacions
app.UseAuthentication();
app.UseAuthorization();
```

3. Crear mètode Seed per a inicialitzar els rols

```
public static async Task CrearRolsInicials(IServiceProvider serviceProvider)
{
    var roleManager = serviceProvider.GetRequiredService<RoleManager<IdentityRole>>();

    string[] rols = { "Admin", "Usuari" };
	foreach (var rol in rols)
	{
		if (!await roleManager.RoleExistsAsync(rol))
        {
            await roleManager.CreateAsync(new IdentityRole(rol));
        }
    }
}
```

4. Afegir els rols als usuaris

```
if (result.Succeeded)
{
    await userManager.AddToRoleAsync(nouAdmin, "Admin");
}
```

5. Afegir les autoritzacions als mètodes

```
[Authorize(Roles = "Periodista,Editor")]
```

### 4. Descriu com i per a què serveix un token, quina informació té i com participa en l’autenticació d’un usuari en un web.

a. **Com es pot saber si un token és vàlid?**
a. **Qui ho verifica el client o el servidor? Per què?**
a. **Quines llibreries fas servir per utilitzar tokens?**

Un token és un codi d'autenticació que serveix per a poder mantenir la sessió oberta d'un usuari. El token conté els atributs amb informació del usuari o del token codificats.

Al iniciar sessió el servidor genera un token amb informació sobre l'usuari i el mateix token, aquest token s'envia a l'usuari per a que aquest tingui accés als serveis.

Un token entre la informació que conté, es troba la data d'expiració, si la data no ha arribat encara el token encara és vàlid, si la data ja ha passat el token ja no és vàlid.

El token el verifica el servidor, perquè l'usuari és qui té el token, i li dona al servidor perquè el verifiqui.

Per a poder utilitzar els tokens és necessaria la llibreria **Microsoft.AspNetCore.Authentication.JwtBearer**.

### 5. Explica les diferències entre encriptació hash, encriptació simètrica i encriptació asimètrica. En quins casos s’utilitzen cadascun?

El hash és tipus de encriptació irreversible, una vegada encriptat amb hash la informació ja no es pot recuperar, això s'utilitza en contrasenyes.

L'encriptació simètrica utilitza una clau per a encriptar, tot aquell que tingui la clau ho podrà desecriptar. Aquesta clau s'utilitza en les VPN.

L'encriptació asimètrica utilitza dues claus, una pública i una privada. El que s'encripti amb la clau pública es desencripta amb la privada. La clau pública és una clau que tothom té accés mentre que la clau privada només el propietari de les claus és qui té accés. Per enviar un missatge aquest s'encripta amb la clau pública del receptor, així aconsequix que només el receptor pugui llegir el missatge amb la seva clau privada. S'utilitza en els correus electònics.

### 6. Aquest codi de creació d’una aplicació web API és correcte? Si no ho és indica per que i quins canvis faries. Afegiries alguna cosa més en cas de ser per l’API de l’exercici 3?

```
var builder = WebApplication.CreateBuilder(args);

builder.Services.AddSwaggerGen();

builder.Services.AddControllers();

builder.Services.AddEndpointsApiExplorer();

app.UseAuthorization();

app.MapControllers(); 

var app = builder.Build();

if (app.Environment.IsDevelopment()) 
{
    app.UseSwagger();
    app.UseSwaggerUI();
}
app.UseHttpsRedirection();


app.Run();
```

En el codi app s'utilitza abans de que sigui declarat.

Falta el **app.UseAuthorization();** en el codi.

Falta el **Service.AddIndentity**.

Per a l'exercici 3 s'haura de afegir la configuració de l'autenticació, l'inicialització dels rols i configurar l'autorització, afegir el DbContext, els controladors.

```
var builder = WebApplication.CreateBuilder(args);

builder.Services.AddSwaggerGen();

builder.Services.AddControllers();

builder.Services.AddEndpointsApiExplorer();

builder.Services.AddAuthentication(JwtBearerDefaults.AuthenticationScheme)
    .AddJwtBearer(options => {
        // config jwt aquí
    });

builder.Services.AddAuthorization(options => {
    options.AddPolicy("SubscriptorOnly", p => p.RequireRole("Subscriptor"));
    options.AddPolicy("PeriodistaOnly", p => p.RequireRole("Periodista"));
    options.AddPolicy("EditorOnly", p => p.RequireRole("Editor"));
});

builder.Services.AddDbContext<AppDbContext>(...);
builder.Services.AddControllers();

var app = builder.Build();

if (app.Environment.IsDevelopment())
{
    app.UseSwagger();
    app.UseSwaggerUI();
}

app.UseHttpsRedirection();
app.UseAuthentication();
app.UseAuthorization();

app.MapControllers();

app.Run();

```

### 7. Determina si el següent codi té errors i si compleix els principis de programació segura. En cas de no fer-ho raona el motiu i quin principi ha vulnerat:

```
[HttpPost("login")]
public async Task<IActionResult> Login([FromBody] UserIdentity user)
{
	//Certifiquem que el mail existeix
	var usuari = await _userManager.FindByEmailAsync(user.Email);
	if (usuari == null)
		return Ok(“Error”);

	var claims = new List<Claim>()
	{
		new Claim(ClaimTypes.Name, usuari.UserName),
		new Claim(ClaimTypes.NameIdentifier, usuari.Id.ToString())
	};

	_logger.Information(“Usuari {usuari.UserName} amb id {usuari.Id.ToString()} i password {usuari.password} ha fet logging amb éxit!”);

	var token = CreateToken(claims.ToArray());
	return Ok(usuari);
}
```

El mètode rep un user, que conté tota l'informació de l'usuari a iniciar sessió, tota aquesta informació no és necessaria per a fer login, només és necessita un identificador, com el correu, i una contrasenya.

Si no troba l'usuari el mètode retorna un Ok, quan hauria de rotornar un NotFound, ja que no ha trobat l'usuari. Això vulnera el principi de **Principi de la mínima exposició**.

Si troba l'usuari el mètode no comprova si la contrasenya d'aquest és correcte, així que no comprova l'identitat del usuari correctament, només comprova que aquest es trobi en la base de dades. Això vulnera el principi de **Autenticació i autorització segura**.

El logger registra tota la informació del usuari, tant el id que té, com la seva contrasenya, aquests dos són completament inecessaris, sobre tot la contrasenya. Això vulnera el principi de **Principi de la mínima exposició**.

Al acabar el mètode aquest retorna l'usuari, i amb això tota la seva informació, això no és corrececte. Això vulnera el principi de **Principi del menor privilegi i mínima exposició**.

Si el que rep el mètode no ha estat validat anteriorment també vulnera el princpi de **Validació i sanejament d'entrada**.