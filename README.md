# AI-102 Developer Guide: Building an AI-Powered Customer Support Assistant in C#

---

## Setting Up Azure AI Foundry Resources

### **Option 1: Using Azure Portal**

1. **Sign in to Azure Portal**: [https://portal.azure.com](https://portal.azure.com)
2. **Create Resource Group**:
   - Navigate to **Resource Groups** > **Create**.
   - Enter name (e.g., `MyAIResourceGroup`) and select region.
3. **Create Azure AI Foundry Resource**:
   - Search for **Azure AI Foundry** in the marketplace.
   - Click **Create** and provide:
     - Subscription, Resource Group, Region.
     - Resource Name (e.g., `MyAIFoundry`).
   - Click **Review + Create**.
4. **Add Cognitive Services**:
   - Navigate to the AI Foundry resource.
   - Add **Language Service**, **Computer Vision**, and **Speech Service**.
   - Configure pricing tier (e.g., `S`) and region.
5. **Configure Azure AD Authentication**:
   - Go to **Azure Active Directory** > **App Registrations**.
   - Register your app and set redirect URI (e.g., `https://localhost/signin-oidc`).
6. **Retrieve Keys and Endpoints**:
   - Navigate to each Cognitive Service resource.
   - Copy **Endpoint** and **Keys** for SDK integration.

---

### **Option 2: Using Azure CLI**

#### Step 1: Create Resource Group
```bash
az group create --name MyAIResourceGroup --location westeurope
```

#### Step 2: Create Azure AI Foundry Resource
```bash
az resource create   --resource-group MyAIResourceGroup   --name MyAIFoundry   --resource-type "Microsoft.AI/foundry"   --location westeurope   --properties '{}'
```

#### Step 3: Create Language Service
```bash
az cognitiveservices account create   --name MyLanguageService   --resource-group MyAIResourceGroup   --kind Language   --sku S   --location westeurope   --yes
```

#### Step 4: Create Computer Vision Service
```bash
az cognitiveservices account create   --name MyVisionService   --resource-group MyAIResourceGroup   --kind ComputerVision   --sku S   --location westeurope   --yes
```

#### Step 5: Create Speech Service
```bash
az cognitiveservices account create   --name MySpeechService   --resource-group MyAIResourceGroup   --kind Speech   --sku S   --location westeurope   --yes
```

#### Step 6: Configure Azure AD Authentication
```bash
az ad app create --display-name "AiSupportAssistant" --reply-urls "https://localhost/signin-oidc"
```

#### Step 7: Retrieve Keys and Endpoints
```bash
az cognitiveservices account keys list --name MyLanguageService --resource-group MyAIResourceGroup
```

![Architecture Diagram](https://github.com/markmorganSNC/ai-powered-customer-support-assistant/blob/main/ProjectArchitecture.png)

---

## Part 1: Roadmap

### AI-102 Exam Objectives Covered
- **Plan and Manage Azure AI Solutions**
- **Implement Computer Vision Solutions**
- **Implement Natural Language Processing Solutions**
- **Implement Conversational AI Solutions**
- **Integrate AI Services into Applications**

### Learning Resources
- [AI-102 Exam](https://learn.microsoft.com/en-us/certifications/exams/ai-102)
- [Azure for .NET Developers](https://learn.microsoft.com/en-us/dotnet/azure/)
- [Azure Language Service](https://learn.microsoft.com/en-us/azure/ai-services/language-service/)
- [Azure Computer Vision](https://learn.microsoft.com/en-us/azure/ai-services/computer-vision/)
- [Azure Speech Service](https://learn.microsoft.com/en-us/azure/ai-services/speech-service/)

---

## Part 2: Project Plan with Code

### Project Overview
Build an ASP.NET Core Web API that integrates:
- **Azure AI Language Question Answering** (Custom Q&A)
- **Computer Vision** (Image Analysis)
- **Speech-to-Text** (Voice Input)
- **Azure AD Authentication**
- **Deployment to Azure App Service**
- **Blazor UI Frontend**

---

### Milestone 1: Setup ASP.NET Core Web API + Azure SDK

**Steps:**
```shell
dotnet new webapi -n AiSupportAssistant
dotnet add package Azure.AI.Language.QuestionAnswering
dotnet add package Azure.AI.Vision
dotnet add package Microsoft.CognitiveServices.Speech
dotnet add package Microsoft.Identity.Web
dotnet add package Microsoft.Identity.Web.UI
```

**Program.cs Configuration:**
```csharp
var builder = WebApplication.CreateBuilder(args);

builder.Services.AddControllers();
builder.Services.AddAuthentication(OpenIdConnectDefaults.AuthenticationScheme)
    .AddMicrosoftIdentityWebApp(builder.Configuration.GetSection("AzureAd"));
builder.Services.AddRazorPages();
builder.Services.AddServerSideBlazor();

builder.Services.Configure<AzureSettings>(builder.Configuration.GetSection("AzureSettings"));

var app = builder.Build();
app.UseAuthentication();
app.UseAuthorization();
app.MapControllers();
app.MapBlazorHub();
app.MapFallbackToPage("/_Host");
app.Run();
```

**AzureSettings.cs:**
```csharp
public class AzureSettings
{
    public string Endpoint { get; set; }
    public string ApiKey { get; set; }
}
```

---

### Milestone 2: Implement Azure AI Language Question Answering

**Controller:**
```csharp
[ApiController]
[Route("api/[controller]")]
[Authorize]
public class QnAController : ControllerBase
{
    private readonly AzureSettings _settings;

    public QnAController(IOptions<AzureSettings> settings)
    {
        _settings = settings.Value;
    }

    [HttpPost("ask")]
    public async Task<IActionResult> AskQuestion([FromBody] string question)
    {
        var client = new QuestionAnsweringClient(new Uri(_settings.Endpoint), new AzureKeyCredential(_settings.ApiKey));
        var response = await client.GetAnswersAsync(question, "your-project-name", "your-deployment-name");
        return Ok(response.Value.Answers.FirstOrDefault()?.Answer);
    }
}
```

---

### Milestone 3: Add Computer Vision Integration

**Controller:**
```csharp
[ApiController]
[Route("api/[controller]")]
[Authorize]
public class VisionController : ControllerBase
{
    private readonly AzureSettings _settings;

    public VisionController(IOptions<AzureSettings> settings)
    {
        _settings = settings.Value;
    }

    [HttpPost("analyze")]
    public async Task<IActionResult> AnalyzeImage([FromBody] string imageUrl)
    {
        var client = new VisionClient(new Uri(_settings.Endpoint), new AzureKeyCredential(_settings.ApiKey));
        var analysis = await client.AnalyzeImageAsync(imageUrl);
        return Ok(analysis.Description.Captions.FirstOrDefault()?.Text);
    }
}
```

---

### Milestone 4: Add Speech-to-Text

**Service Class:**
```csharp
public class SpeechService
{
    private readonly AzureSettings _settings;

    public SpeechService(IOptions<AzureSettings> settings)
    {
        _settings = settings.Value;
    }

    public async Task<string> ConvertSpeechToTextAsync(string audioFilePath)
    {
        var config = SpeechConfig.FromSubscription(_settings.ApiKey, "your-region");
        using var recognizer = new SpeechRecognizer(config, audioFilePath);
        var result = await recognizer.RecognizeOnceAsync();
        return result.Text;
    }
}
```

---

### Milestone 5: Secure with Azure AD
- Configure **Azure AD** in `appsettings.json`:
```json
{
  "AzureAd": {
    "Instance": "https://login.microsoftonline.com/",
    "Domain": "your-domain.onmicrosoft.com",
    "TenantId": "your-tenant-id",
    "ClientId": "your-client-id",
    "CallbackPath": "/signin-oidc"
  }
}
```
- Use `[Authorize]` attribute on controllers.

---

### Milestone 6: Add Blazor UI

**Steps:**
1. Create a Blazor Server project or add Blazor to existing solution:
```shell
dotnet new blazorserver -n AiSupportAssistant.UI
```
2. Add pages for Q&A and Vision:
```razor
@page "/qna"
<h3>Ask a Question</h3>
<input @bind="question" />
<button @onclick="AskQuestion">Ask</button>
<p>@answer</p>

@code {
    private string question;
    private string answer;

    private async Task AskQuestion()
    {
        var http = new HttpClient();
        var response = await http.PostAsJsonAsync("/api/qna/ask", question);
        answer = await response.Content.ReadAsStringAsync();
    }
}
```

---

### Milestone 7: Deploy to Azure App Service + Monitoring
**Steps:**
```shell
dotnet publish -c Release
```
- Deploy to Azure App Service.
- Enable **Application Insights** for monitoring.

---

## ✅ Next Steps
- Implement **Logging & Error Handling**.
- Expand Q&A with **Custom Knowledge Base** in Azure Language Service.
- Add **Role-Based Access Control** for Blazor UI.

---
