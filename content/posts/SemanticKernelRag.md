+++
date = '2025-03-15T11:22:29-04:00'
draft = false
title = 'SemanticKernelRag'
tags = ['programming', 'ai']
categories = ['programming', 'artificial-intelligence']
+++

Okay, let's break down how to use Semantic Kernel with Retrieval Augmented Generation (RAG) in a Blazor application. It can be a bit tricky to get all the pieces working together, so we'll go step by step.

**Understanding the Core Concepts**

1.  **Semantic Kernel:** This is a framework that helps you build intelligent applications by integrating Large Language Models (LLMs) like OpenAI, Azure OpenAI, etc. It provides abstractions for things like plugins, memory, and planners.
2.  **RAG (Retrieval Augmented Generation):** This approach improves LLM responses by first retrieving relevant information from a data source (like a vector database) and then using that information as context when generating the final output. This helps the LLM ground its responses in factual data and reduce hallucinations.
3.  **Blazor:** This is a framework for building interactive web UIs with C# instead of JavaScript. We'll use it to create the front-end for your RAG-powered application.

**High-Level Steps**

Here's a general outline of what we'll need to do:

1.  **Set up your environment:** Install the necessary NuGet packages and configure your API keys.
2.  **Prepare your data source:** Choose a vector database (or similar) and index your data.
3.  **Create Semantic Kernel components:** Build your skills (plugins), memory, and kernel.
4.  **Develop your Blazor UI:** Create a simple interface for users to ask questions.
5.  **Connect Semantic Kernel with Blazor:** Send user input to the kernel, get the RAG-enhanced response, and display it in the UI.

**Let's Get Started with the Code**

I'll provide code snippets with explanations and comments. This will be for a simple Blazor Server app for demonstration purposes.

**1. Setting up your Environment**

*   **Create a New Blazor Server Project:**
    *   In Visual Studio or using the .NET CLI:
        ```bash
        dotnet new blazorserver -o MyRAGApp
        cd MyRAGApp
        ```
*   **Install NuGet Packages:**
    ```bash
    dotnet add package Microsoft.SemanticKernel
    dotnet add package Microsoft.SemanticKernel.Connectors.Memory.Qdrant
    dotnet add package Microsoft.SemanticKernel.Connectors.OpenAI
    ```
    (Choose `Qdrant` or a memory connector of your preference)
    *   Make sure you have `System.Text.Json` package included as well.

**2. Configure App Settings (appsettings.json)**

```json
{
  "Logging": {
    "LogLevel": {
      "Default": "Information",
      "Microsoft.AspNetCore": "Warning"
    }
  },
  "AllowedHosts": "*",
    "SemanticKernel": {
        "OpenAI": {
            "ApiKey": "YOUR_OPENAI_API_KEY",
            "OrgId": "YOUR_OPTIONAL_OPENAI_ORG_ID",
            "ModelId": "gpt-3.5-turbo",
            "EmbeddingModelId": "text-embedding-ada-002"
        },
        "Qdrant":{
            "Endpoint":"http://localhost:6333",
            "VectorSize":1536,
            "CollectionName":"ragcollection"
        }
    }
}
```
* Replace the placeholder with the api key of your preferred LLM API, e.g., OpenAI. Make sure you also replace the placeholder with the correct values of your memory vector database.

**3. Create Data Loader (optional)**

*   For simplicity, let's assume you have a simple text file of data to be indexed, say `knowledge_base.txt`.
*   **Create a DataLoader service (`Services/DataLoader.cs`):**

```csharp
using System.Text;
using Microsoft.SemanticKernel.Memory;
using Microsoft.Extensions.Options;
using Microsoft.SemanticKernel;
using Microsoft.SemanticKernel.Embeddings;
using Microsoft.SemanticKernel.Connectors.Memory.Qdrant;

namespace MyRAGApp.Services
{
    public class DataLoader
    {
        private readonly IOptions<SemanticKernelOptions> _options;
        private readonly ILogger<DataLoader> _logger;
        private readonly ITextEmbeddingGeneration _textEmbeddingGeneration;

        public DataLoader(
            IOptions<SemanticKernelOptions> options,
            ILogger<DataLoader> logger,
            ITextEmbeddingGeneration textEmbeddingGeneration
        )
        {
            _options = options;
            _logger = logger;
            _textEmbeddingGeneration = textEmbeddingGeneration;
        }

        public async Task LoadKnowledge(string filePath, IKernel kernel)
        {
            try
            {
                // Read content from text file
                var content = await File.ReadAllTextAsync(filePath, Encoding.UTF8);
                var lines = content.Split(Environment.NewLine, StringSplitOptions.RemoveEmptyEntries);

                var memoryStore = GetMemoryStore();
                if(memoryStore != null)
                {
                    foreach (var line in lines)
                    {
                        var id = Guid.NewGuid().ToString();
                        await kernel.Memory.SaveInformationAsync(_options.Value.Qdrant.CollectionName, line, id, line, null, _textEmbeddingGeneration);
                    }

                    _logger.LogInformation("Knowledge base loaded successfully");
                } else
                {
                    _logger.LogError("Memory store is null. Cannot load the knowledge base");
                }
            }
            catch (Exception ex)
            {
                _logger.LogError($"Error reading or loading data: {ex.Message}");
            }
        }

        private ISemanticTextMemory? GetMemoryStore()
        {
            if (_options.Value.Qdrant is not null)
            {
                return new QdrantMemoryStore(
                    _options.Value.Qdrant.Endpoint,
                    _options.Value.Qdrant.VectorSize
                );
            }

            return null;
        }
    }
}
```
*  Add the corresponding SemanticKernelOptions class:
```csharp
using System.ComponentModel.DataAnnotations;

namespace MyRAGApp
{
    public class SemanticKernelOptions
    {
        [Required]
        public OpenAIConfig? OpenAI { get; set; }

        public QdrantConfig? Qdrant { get; set; }

    }
    public class OpenAIConfig
    {
        [Required]
        public string? ApiKey { get; set; }

        public string? OrgId { get; set; }

        [Required]
        public string? ModelId { get; set; }
        [Required]
        public string? EmbeddingModelId { get; set; }
    }
    public class QdrantConfig
    {
        [Required]
        public string? Endpoint { get; set; }
        [Required]
        public int? VectorSize { get; set; }
        [Required]
        public string? CollectionName { get; set; }
    }
}
```
*   Make sure to add your knowledge base content in the `knowledge_base.txt` in the `wwwroot/` directory or other preferred location.

**4. Kernel Setup**

*   **Create a Kernel service (`Services/KernelService.cs`):**
```csharp
using Microsoft.SemanticKernel;
using Microsoft.SemanticKernel.Connectors.OpenAI;
using Microsoft.Extensions.Options;
using Microsoft.SemanticKernel.Memory;
using Microsoft.SemanticKernel.Embeddings;
using Microsoft.SemanticKernel.Connectors.Memory.Qdrant;

namespace MyRAGApp.Services
{
    public class KernelService
    {
        private readonly IOptions<SemanticKernelOptions> _options;
        private readonly ILogger<KernelService> _logger;

        public KernelService(IOptions<SemanticKernelOptions> options, ILogger<KernelService> logger)
        {
            _options = options;
            _logger = logger;
        }

        public IKernel GetKernel()
        {
            try
            {
                var builder = Kernel.Builder;

                var apiKey = _options.Value.OpenAI?.ApiKey;
                var orgId = _options.Value.OpenAI?.OrgId;
                var modelId = _options.Value.OpenAI?.ModelId;
                var embeddingModelId = _options.Value.OpenAI?.EmbeddingModelId;

                if (string.IsNullOrEmpty(apiKey) || string.IsNullOrEmpty(modelId) || string.IsNullOrEmpty(embeddingModelId))
                {
                    _logger.LogError("OpenAI API settings are not correctly configured.");
                    return null;
                }

                builder.AddAzureOpenAIChatCompletion(modelId, apiKey,orgId);
                builder.AddAzureOpenAITextEmbeddingGeneration(embeddingModelId, apiKey, orgId);
                builder.Services.AddTransient((sp) => GetMemoryStore());
                return builder.Build();
            }
            catch (Exception ex)
            {
                _logger.LogError($"Error creating Kernel: {ex.Message}");
                return null;
            }
        }

        private ISemanticTextMemory? GetMemoryStore()
        {
            if (_options.Value.Qdrant is not null)
            {
                return new QdrantMemoryStore(
                    _options.Value.Qdrant.Endpoint,
                    _options.Value.Qdrant.VectorSize
                );
            }

            return null;
        }
    }
}
```

**5. Register Services in `Program.cs`**

```csharp
using MyRAGApp;
using MyRAGApp.Services;

var builder = WebApplication.CreateBuilder(args);

builder.Services.Configure<SemanticKernelOptions>(builder.Configuration.GetSection("SemanticKernel"));
// Add services to the container.
builder.Services.AddRazorPages();
builder.Services.AddServerSideBlazor();
builder.Services.AddSingleton<KernelService>();
builder.Services.AddSingleton<DataLoader>();

var app = builder.Build();

// Configure the HTTP request pipeline.
if (!app.Environment.IsDevelopment())
{
    app.UseExceptionHandler("/Error");
    app.UseHsts();
}

app.UseHttpsRedirection();
app.UseStaticFiles();
app.UseRouting();
app.MapBlazorHub();
app.MapFallbackToPage("/_Host");


using (var scope = app.Services.CreateScope())
{
    var dataLoader = scope.ServiceProvider.GetRequiredService<DataLoader>();
    var kernelService = scope.ServiceProvider.GetRequiredService<KernelService>();
    var kernel = kernelService.GetKernel();
    var filePath = Path.Combine(app.Environment.WebRootPath, "knowledge_base.txt");

    if (kernel is not null)
    {
        await dataLoader.LoadKnowledge(filePath, kernel);
    }
}

app.Run();
```

**6. Blazor UI**

*   **Modify `Pages/Index.razor`:**

```razor
@page "/"
@using MyRAGApp.Services
@inject KernelService KernelService
@inject ILogger<Index> Logger
@inject DataLoader DataLoader

<h1>RAG with Semantic Kernel</h1>

<div class="input-group mb-3">
    <input type="text" class="form-control" placeholder="Ask a question" @bind="UserQuestion" />
    <button class="btn btn-primary" @onclick="AskQuestion">Ask</button>
</div>

@if (!string.IsNullOrEmpty(Response))
{
    <div class="alert alert-secondary">
        @Response
    </div>
}

@code {
    private string UserQuestion { get; set; } = string.Empty;
    private string Response { get; set; } = string.Empty;

    private async Task AskQuestion()
    {
        if (string.IsNullOrEmpty(UserQuestion)) return;

         var kernel = KernelService.GetKernel();

        if (kernel is null)
        {
            Logger.LogError("Kernel was not properly initialized");
            Response = "Error initializing the kernel.";
            return;
        }

        try
        {
           var result = await kernel.Memory.SearchAsync("ragcollection", UserQuestion, limit:1, minRelevanceScore:0.7);

             var context = result.FirstOrDefault()?.Text;

             if(context != null)
            {
                var prompt = @$"You are an AI assistant. Use the provided context to answer the question. If the context is not relevant for the answer, say you don't know. 

                    Context:
                    {context}

                    Question: {UserQuestion}
                ";

                var promptTemplateFactory = new Microsoft.SemanticKernel.PromptTemplate.PromptTemplateFactory();
                var promptTemplate = promptTemplateFactory.Create(prompt);

                var chatFunction = kernel.CreateSemanticFunction(promptTemplate);
                var answer = await kernel.RunAsync(new KernelArguments(), chatFunction);

                Response = answer.ToString();
            } else
             {
                Response = "No relevant knowledge found. Please ask a question related to the content of knowledge base file";
             }
            

        }
        catch (Exception ex)
        {
             Logger.LogError($"Error processing question: {ex.Message}");
             Response = $"An error occurred: {ex.Message}";
        }
    }
}
```

**Explanation:**

1.  **Dependency Injection:** We inject `KernelService` and `DataLoader` to use them in the page.
2.  **UI Elements:**
    *   A text input (`UserQuestion`) for the user to type their question.
    *   A button to trigger the question processing.
    *   A div (`Response`) to display the result.
3.  **`AskQuestion` Method:**
    *   Gets the kernel from the `KernelService`.
    *   Uses Semantic Kernel's memory to search for relevant data.
    *   Constructs an advanced prompt with retrieved context.
    *   Uses Semantic Kernel to run an LLM completion and get the response.
    *   Updates the `Response` variable, which triggers a UI update.

**How to Run**

1.  Make sure that your Qdrant instance is running.
2.  Set your API keys in `appsettings.json`.
3.  Add your knowledge base content in the `knowledge_base.txt` file.
4.  Build and run your project (`dotnet run`).
5.  Navigate to the application in your browser (`https://localhost:7085` by default).
6.  Type your question in the input and hit "Ask."

**Important Considerations**

*   **Error Handling:** Implement better error handling and logging throughout your application.
*   **Memory Store Selection:** Explore other vector databases or memory stores (like Azure Cognitive Search). `Qdrant` is a good starting point.
*   **Prompt Engineering:** Fine-tune your prompts to get the best results.
*   **Performance:** Vector search and LLM completions can be slow. Optimize based on your application's needs.
*   **Security:** Be mindful of storing your API keys securely. Use environment variables or secrets management.

Let me know if you would like to dive deeper into specific parts of this process, like more advanced prompt engineering or using a different memory store. I'm here to help you get your RAG application working!
