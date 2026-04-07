AI Research Assistant: Local RAG Stack
A high-performance research tool built with Svelte 5 and a Model Context Protocol (MCP) backend. This application performs real-time web scraping via DuckDuckGo and uses MediaPipe to rank the most relevant information for your queries.

Project Architecture (Part 3)
The "AI Tech Stack" consists of three primary layers:

Frontend: SvelteKit using Runes ($state) for reactive UI management.

Proxy Layer: mcpo (MCP Proxy) converting MCP tool protocols into standard REST API endpoints.

RAG Engine: A Dockerized Python backend that handles:

Retrieval: Searching DuckDuckGo for live data.

Augmentation: Scraping full text from discovered URLs.

Ranking: Using MediaPipe embeddings to calculate relevance scores. (Doesn't work currently)

Setup & Installation
1. Backend (Docker)
Ensure your Docker container is running and exposing port 9001:

Bash
# Build the image
docker build -t local-rag-proxy .

# Run the container
docker run -d -p 9001:8000 --name rag-proxy-app local-rag-proxy
2. Frontend (SvelteKit)
Initialize your project: npm install

Start the development server: npm run dev

Open http://localhost:5173 in your browser.

🛠️ UI/UX Design (Part 4)
The interface is designed for Technical Research, focusing on:

Evidence-First Layout: Raw search snippets are displayed alongside relevance scores to build user trust.

Relevance Badges: Visual indicators that show if the result matches

Async Feedback: Integrated loading states to manage the 10-15 second scraping and ranking window.

🔍 Troubleshooting
"Failed to parse JSONRPC message" (Terminal Logs)
If you see ValidationError or Invalid JSON in your terminal logs (as seen in your recent sessions), this is expected behavior with the current mcp-local-rag implementation.

Cause: The server prints "Status Updates" (e.g., Fetched https://...) to stdout. The proxy tries to read these as JSON-RPC messages and fails.

Impact: None. The proxy successfully ignores these text strings and captures the actual JSON data at the end of the process.

All Badges show 0%
If results appear but relevance is 0%:

Verify the score field exists in the API response (Network Tab -> Response).

Check if the MediaPipe model initialized correctly in the Docker logs.

Ensure your query is specific enough for the embedding model to find a match in the text.

📝 LLM Prompt Design
To complete the "Synthesis" part of the stack, use this prompt template with an LLM:

System Prompt: "You are a Research Assistant. I will provide you with a 'context' block containing search results.Your Task: Summarize the answer in 3 clear paragraphs.Citation Rule: If a fact comes from a specific URL provided in the context, add a clickable markdown link at the end of the sentence. "

Key Commands
View Logs: docker logs -f rag-proxy-app

Restart Backend: docker restart rag-proxy-app

Stop Backend: docker stop rag-proxy-app