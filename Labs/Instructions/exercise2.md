# Exercise 2: Exploring the Building Blocks of the Application [Read-Only]

### Estimated Duration: 30 Minutes

## Lab Scenario

In this exercise, you will review the source code of the Creative Writer application and familiarize yourself with the technologies used. You will learn about the FastAPI framework, how real-time data streaming is handled, and how monitoring and observability tools like OpenTelemetry are integrated. By the end of this exercise, you will have a solid understanding of the code structure and the different tech stacks that power the application.

## Lab Objectives

After you complete this exercise, you will be able to:

- Know the Technology Stacks Used
- Review Source Code Files

### Task 1: Know the Technology Stacks Used 

In this task, you will familiarize yourself with the various technologies that power the Contoso Creative Writer application. The app leverages a combination of powerful tools and frameworks to help you write well-researched, product-specific articles. As you explore the code, you will learn about the following key technology stacks:

- **FastAPI Framework**

  - ***What It Is:*** FastAPI is a modern, fast (high-performance) web framework for building APIs with Python 3.7+ based on standard Python-type hints.

  - ***Why It Is Used:*** FastAPI is chosen for its performance and ease of use. It allows you to quickly create a RESTful API for interacting with the various agents involved in the article-writing process.

  - ***Reference:*** [FastAPI](https://fastapi.tiangolo.com/)

- **Prompty**

  - ***What It Is:*** Prompty is a library that allows you to manage, create, and evaluate prompts for language models like GPT-3. It simplifies the process of interacting with AI models by providing better prompt management tools.
    
  - ***Why It Is Used:*** Prompty helps in organizing and refining prompts to guide AI agents effectively, ensuring that each step in the article creation process (research, writing, editing) is powered by accurate and relevant inputs.

  - ***Reference:*** [Prompty](https://prompty.ai/)

- **Bing Search API**

  - ***What It Is:*** The Bing Search API allows you to programmatically access Bing search results. It’s useful for gathering information from the web, such as research materials related to the article topic.

  - ***Why It Is Used:*** In the Contoso Creative Writer app, the Bing Search API is used by the research agent to gather relevant information about the topic provided by the user.

  - ***Reference:*** [Bing Search API](https://www.microsoft.com/en-us/bing/apis)

- **Azure OpenAI**

  - ***What It Is:*** Azure OpenAI provides access to OpenAI's powerful models, including GPT-3, which can be used for a variety of tasks, such as natural language understanding, generation, and more.

  - ***Why It Is Used:*** In this app, Azure OpenAI powers the various AI agents that handle tasks like research, product matching, article writing, and editing.

  - ***Reference:*** [Azure OpenAI](https://azure.microsoft.com/en-us/products/ai-services/openai-service)

- **Azure AI Search**

  - ***What It Is:*** Azure AI Search (formerly known as Azure Cognitive Search) is a cloud search service with built-in AI capabilities to help you extract insights from data, such as searching for related products using semantic similarity.

  - ***Why It Is Used:*** The product agent uses Azure AI Search to find products related to the research topic by performing a semantic similarity search in a vector store, improving the relevance of product recommendations.

  - ***Reference:*** [Azure AI Search](https://learn.microsoft.com/en-us/azure/search/search-what-is-azure-search)

### Task 2: Review Source Code Files 

In this task, you will review three core code files that together initialize a FastAPI application with tracing and orchestration capabilities. You will analyze the main application setup, task flow with agents, and tracing configuration using OpenTelemetry and Azure Monitor, gaining insight into API handling, task management, and monitoring setup.

1. From the **Explorer** menu of **Visual Studio Code**, navigate to the `/src/api/main.py` file and review the codes.

1. The first part of the file imports various libraries and modules used throughout the application.

   ```python
   import os
   from pathlib import Path
   from fastapi import FastAPI
   from dotenv import load_dotenv
   from prompty.tracer import trace
   from prompty.core import PromptyStream, AsyncPromptyStream
   from fastapi.responses import StreamingResponse
   from fastapi.middleware.cors import CORSMiddleware
   from opentelemetry.instrumentation.fastapi import FastAPIInstrumentor
   from tracing import init_tracing
   from orchestrator import Task, create
   ```
   
   >**OS and Path:** Used for file path handling and environment variable management.

   >**FastAPI:** The web framework used to build the API.

   >**Dotenv:** Loads environment variables from a .env file.

   >**Prompty:** A library for managing and evaluating prompts with OpenAI.

   >**StreamingResponse:** Used for streaming data from the server to the client.

   >**CORSMiddleware:** Middleware to handle Cross-Origin Resource Sharing (CORS).

   >**FastAPIInstrumentor:** Part of OpenTelemetry for instrumenting the FastAPI app with tracing.

   >**Tracing and orchestrator:** Custom modules for tracing setup and task orchestration.

1. The next part of the file focuses on loading environment variables and initializing the **FastAPI** application.

   ```python
   base = Path(__file__).resolve().parent
   load_dotenv()
   tracer = init_tracing()
   app = FastAPI()
   ```
   >**load_dotenv():** This function loads the environment variables from a .env file into the Python environment. It ensures that sensitive information like API keys and database credentials are stored securely and can be accessed within the app.

   >**init_tracing():** Initializes the tracing system for monitoring and debugging, ensuring that requests and interactions can be traced across services.

   >**FastAPI():** This line initializes the FastAPI app, which will handle HTTP requests and route them to the appropriate functions.

1. In the next part, there is a setup of  CORS (Cross-Origin Resource Sharing) middleware.

   ```python
   code_space = os.getenv("CODESPACE_NAME")
   app_insights = os.getenv("APPINSIGHTS_CONNECTIONSTRING")

   if code_space: 
       origin_8000= f"https://{code_space}-8000.app.github.dev"
       origin_5173 = f"https://{code_space}-5173.app.github.dev"
       ingestion_endpoint = app_insights.split(';')[1].split('=')[1]
    
       origins = [origin_8000, origin_5173, os.getenv("API_SERVICE_ACA_URI"), os.getenv("WEB_SERVICE_ACA_URI"), ingestion_endpoint]
   else:
       origins = [
           o.strip()
           for o in Path(Path(__file__).parent / "origins.txt").read_text().splitlines()
       ]
       origins = ['*']
   ```

   >**CORS Middleware** is used to allow or restrict cross-origin requests (requests coming from other domains).

1. The next part creates the API endpoint for handling article creation in the application.

   ```python
   @app.post("/api/article")
   @trace
   async def create_article(task: Task):
       return StreamingResponse(
           PromptyStream(
               "create_article", create(task.research, task.products, task.assignment)
           ),
           media_type="text/event-stream",
       )
   ```

   >**POST/API/Article:** This endpoint accepts a POST request to create an article based on the task details. It uses the PromptyStream to handle real-time streaming of the article creation process.
  
1. As you have reviewed `main.py`. Now, select `orchestrator.py` from the left **Explorer** menu. This file is responsible for structured logging, generating, refining, and evaluating articles.

1. Navigate to the `create` function, which is the core orchestrator in this code, managing the flow between various agents to produce an article.

   ```python
    @trace
    def create(research_context, product_context, assignment_context, evaluate=True):
        feedback = "No Feedback"

        # Research Agent Task
        yield start_message("researcher")
        research_result = researcher.research(research_context, feedback)
        yield complete_message("researcher", research_result)

        # Marketing/Product Agent Task
        yield start_message("marketing")
        product_result = product.find_products(product_context)
        yield complete_message("marketing", product_result)

        # Writing Agent Task
        yield start_message("writer")
        yield complete_message("writer", {"start": True})
        writer_result = writer.write(research_context, research_result, product_context, product_result, assignment_context, feedback)

        # Processing and Editor Feedback
        full_result = " "
        for item in writer_result:
            full_result = full_result + f'{item}'
            yield complete_message("partial", {"text": item})
        processed_writer_result = writer.process(full_result)
        
        # Editor Agent Task
        yield start_message("editor")
        editor_response = editor.edit(processed_writer_result['article'], processed_writer_result["feedback"])
        yield complete_message("editor", editor_response)
        yield complete_message("writer", {"complete": True})
   ```

   >The `@trace` decorator traces function execution, likely capturing each step in distributed tracing tools.

   >**Researcher Agent:** Start by invoking the researcher agent with a research_context, gathering topic-specific data. Each agent stage logs a starting and completion message.

   >Use the `product` agent to find relevant products, contributing contextual content for the article. The `writer` agent combines research, product information, and the assignment context to draft the article.

   >After initial writing, the `editor` agent reviews the draft. If it doesn’t meet quality standards, the editor sends feedback to improve content in a loop. Each step yields a `Message` instance to communicate progress, which can be streamed in real-time to a client or logging system.

1. As you review `orchestartor.py`, navigate to the `tracing.py` file from the **Explorer** menu. This file helps to trace all the operations and send data for logging and monitoring.

1. In the `tracing.py` file, find the `init_tracing` function, which is a crucial part of the file.

   ```python
   def init_tracing(local_tracing: bool = False):
    if local_tracing:
        # Use PromptyTracer for local tracing
        local_trace = PromptyTracer()
        Tracer.add("PromptyTracer", local_trace.tracer)
    else:
        # Use OpenTelemetry tracer with Azure Monitor for remote tracing
        app_insights = os.getenv("APPINSIGHTS_CONNECTIONSTRING")
        oteltrace.set_tracer_provider(TracerProvider(sampler=ParentBasedTraceIdRatio(1.0)))
        oteltrace.get_tracer_provider().add_span_processor(
            BatchSpanProcessor(AzureMonitorTraceExporter(connection_string=app_insights))
        )
        return oteltrace.get_tracer(_tracer)
   ```

   > **Local vs. Remote Tracing:** local_tracing determines if the tracing uses a local `PromptyTracer` (useful for debugging without external dependencies) or OpenTelemetry with Azure Monitor, enabling insights on Azure’s Application Insights.

   >**Azure Monitor Integration:** The `AzureMonitorTraceExporter` sends trace data to Azure, providing remote monitoring of spans and trace data for performance and diagnostics.

## Summary

In this exercise, you have reviewed and analyzed three key code files that collectively establish a FastAPI application with integrated task orchestration and tracing. You explored the application’s core setup, examined the workflow of agents that handle different task components, and assessed the tracing configuration using OpenTelemetry and Azure Monitor. This review provided an understanding of how the application handles API requests, manages task flows, and sets up monitoring for performance insights and error tracking.

### You have successfully completed this exercise!!



