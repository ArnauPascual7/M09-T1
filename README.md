# M09-T1

Arnau Pascual

## Principis de Programaci� segura

### Principi de menor privilegi

Cada component, proc�s o usuari nom�s ha de tenir els permisos m�nims necessaris per fer la seva tasca.

### Validaci� i sanejament d'entrada

Tota entrada s'ha de validar i netejar per evitar injeccions, errors o abusos.

### Principi de la m�nima exposici�

No mostrar ni retornar m�s informaci� de la necess�ria.

### Principi d'autenticaci� i autoritzaci�s segura

Verificar correctament la identitat i assegurar que cada acci� nom�s la pugui fer qui estigui autoritzat.

### An�lisis, proves i auditories de seguretat

**Aquest no �s un privilegi, per� si una pr�ctica de seguretat recomanada.**

Revisar i provar el codi per detectar vulnerablitats.

## Exercicis

### 1. Quina �s la difer�ncia entre un socket i un websocket? Quin dels dos pot ser m�s segur? Quines implementacions poden fer-lo m�s segur?

Un socket �s una connexi� entre dos sistemes, una interf�cie bidireccional entre el client i el servidor mitjan�ant els protocols TCP i UDP.

Un websocket �s una connexi� bidireccional i persistents entre el client i el servidor amb una �nica connexi� TCP, aquesta est� pensada per a navgeadors i entorns web.

Els websockets s�n m�s segurs, ja que per defecte utilitzen https.

Per a fer els websockets m�s segurs, es pot autenticar l'usuari abans d'establir la connexi� a m�s de posar l'imits de temps a la connexi�.

### 2. Descriu els passos a realitzar amb SignalR un sistema per veure els valors de la borsa en temps real. No s�ha d�implementar.

1. Instal�lar llibreries als projectes client i servidor de SignalR.
1. Definir la classe Hub
1. Configurar al servidor amb els m�todes d'enviament de missatges.
1. Configurar al client i implementar amb js la rebuda dels missatges.

### 3. Un diari local vol publicar les seves not�cies al seu web. Les not�cies es publiquen per categories. Un usuari del web pot veure els titulars i la data de les not�cies. Si vol llegir l�article sencer (titular, subt�tol, data i hora, redacci�, autor i imatge de portada) ha d'autenticar-se com a subscriptor. Els periodistes, pr�viament identificats, redacten les not�cies i les inserten a la base de dades a trav�s d�un formulari. Una not�cia no es publicar� fins que un editor la revisa i la valida. L�editor pot modificar l�article.

**Desenvolupes una APIrest que gestiona les not�cies i els usuaris.**
- **Implementa les entitats necess�ries i les dades de cada una.**
- **Descriu els rols d�usuari i descriu la pol�tica de seguretat de privilegis que t� cada rol.**
- **Defineix la classe del controlador que gestiona les not�cies i els m�todes. Assegurat que compleix els requisits d�autoritzaci� necessaris. Defineix de manera �ptima els endpoints i els m�todes HTTP. Fes servir els DTO�s que consideris.**
- **Escriu com hauria de ser el missatge JSON del body del response per afegir una nova not�cia.**
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

Tothom: No �s un rol real que s'hagui de crear, per� t� permisos. Pot veure els titulars i dates de tots els articles

Subscriptor: Pot llegir amb detall tots els articles

Periodista: Pot insertar articles, per� no es mostren fins que un Editor ho comprovi

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
    //�
}
```

2. Configurar el Program.cs, afegir IdentityRole i carregar els rols inicials

```
builder.Services.AddIdentity<ApplicationUser, IdentityRole>()
    .AddEntityFrameworkStores<ApplicationDbContext>()
    .AddDefaultTokenProviders();

//�

var app = builder.Build();

//�

//Carreguem els rols inicials
using (var scope = app.Services.CreateScope())
{
    var services = scope.ServiceProvider;
    await CrearRolsInicials(services);
}

//�

//Afegim el middleware d�autoritzacions i d�autenticacions
app.UseAuthentication();
app.UseAuthorization();
```

3. Crear m�tode Seed per a inicialitzar els rols

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

5. Afegir les autoritzacions als m�todes

```
[Authorize(Roles = "Periodista,Editor")]
```

### 4. Descriu com i per a qu� serveix un token, quina informaci� t� i com participa en l�autenticaci� d�un usuari en un web.

a. **Com es pot saber si un token �s v�lid?**
a. **Qui ho verifica el client o el servidor? Per qu�?**
a. **Quines llibreries fas servir per utilitzar tokens?**

Un token �s un codi d'autenticaci� que serveix per a poder mantenir la sessi� oberta d'un usuari. El token cont� els atributs amb informaci� del usuari o del token codificats.

Al iniciar sessi� el servidor genera un token amb informaci� sobre l'usuari i el mateix token, aquest token s'envia a l'usuari per a que aquest tingui acc�s als serveis.

Un token entre la informaci� que cont�, es troba la data d'expiraci�, si la data no ha arribat encara el token encara �s v�lid, si la data ja ha passat el token ja no �s v�lid.

El token el verifica el servidor, perqu� l'usuari �s qui t� el token, i li dona al servidor perqu� el verifiqui.

Per a poder utilitzar els tokens �s necessaria la llibreria **Microsoft.AspNetCore.Authentication.JwtBearer**.

### 5. Explica les difer�ncies entre encriptaci� hash, encriptaci� sim�trica i encriptaci� asim�trica. En quins casos s�utilitzen cadascun?

El hash �s tipus de encriptaci� irreversible, una vegada encriptat amb hash la informaci� ja no es pot recuperar, aix� s'utilitza en contrasenyes.

L'encriptaci� sim�trica utilitza una clau per a encriptar, tot aquell que tingui la clau ho podr� desecriptar. Aquesta clau s'utilitza en les VPN.

L'encriptaci� asim�trica utilitza dues claus, una p�blica i una privada. El que s'encripti amb la clau p�blica es desencripta amb la privada. La clau p�blica �s una clau que tothom t� acc�s mentre que la clau privada nom�s el propietari de les claus �s qui t� acc�s. Per enviar un missatge aquest s'encripta amb la clau p�blica del receptor, aix� aconsequix que nom�s el receptor pugui llegir el missatge amb la seva clau privada. S'utilitza en els correus elect�nics.

### 6. Aquest codi de creaci� d�una aplicaci� web API �s correcte? Si no ho �s indica per que i quins canvis faries. Afegiries alguna cosa m�s en cas de ser per l�API de l�exercici 3?

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

Per a l'exercici 3 s'haura de afegir la configuraci� de l'autenticaci�, l'inicialitzaci� dels rols i configurar l'autoritzaci�, afegir el DbContext, els controladors.

```
var builder = WebApplication.CreateBuilder(args);

builder.Services.AddSwaggerGen();

builder.Services.AddControllers();

builder.Services.AddEndpointsApiExplorer();

builder.Services.AddAuthentication(JwtBearerDefaults.AuthenticationScheme)
    .AddJwtBearer(options => {
        // config jwt aqu�
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

### 7. Determina si el seg�ent codi t� errors i si compleix els principis de programaci� segura. En cas de no fer-ho raona el motiu i quin principi ha vulnerat:

```
[HttpPost("login")]
public async Task<IActionResult> Login([FromBody] UserIdentity user)
{
	//Certifiquem que el mail existeix
	var usuari = await _userManager.FindByEmailAsync(user.Email);
	if (usuari == null)
		return Ok(�Error�);

	var claims = new List<Claim>()
	{
		new Claim(ClaimTypes.Name, usuari.UserName),
		new Claim(ClaimTypes.NameIdentifier, usuari.Id.ToString())
	};

	_logger.Information(�Usuari {usuari.UserName} amb id {usuari.Id.ToString()} i password {usuari.password} ha fet logging amb �xit!�);

	var token = CreateToken(claims.ToArray());
	return Ok(usuari);
}
```

El m�tode rep un user, que cont� tota l'informaci� de l'usuari a iniciar sessi�, tota aquesta informaci� no �s necessaria per a fer login, nom�s �s necessita un identificador, com el correu, i una contrasenya.

Si no troba l'usuari el m�tode retorna un Ok, quan hauria de rotornar un NotFound, ja que no ha trobat l'usuari. Aix� vulnera el principi de **Principi de la m�nima exposici�**.

Si troba l'usuari el m�tode no comprova si la contrasenya d'aquest �s correcte, aix� que no comprova l'identitat del usuari correctament, nom�s comprova que aquest es trobi en la base de dades. Aix� vulnera el principi de **Autenticaci� i autoritzaci� segura**.

El logger registra tota la informaci� del usuari, tant el id que t�, com la seva contrasenya, aquests dos s�n completament inecessaris, sobre tot la contrasenya. Aix� vulnera el principi de **Principi de la m�nima exposici�**.

Al acabar el m�tode aquest retorna l'usuari, i amb aix� tota la seva informaci�, aix� no �s corrececte. Aix� vulnera el principi de **Principi del menor privilegi i m�nima exposici�**.

Si el que rep el m�tode no ha estat validat anteriorment tamb� vulnera el princpi de **Validaci� i sanejament d'entrada**.