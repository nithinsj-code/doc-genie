# DocGenius

DocGenius is an AI-powered document analysis and content generation web application. It allows users to upload PDF documents, extract their content, and interactively ask questions about the document using Google Gemini AI. The application also provides an AI Content Generator for creating additional content based on user prompts.

## Features

- **Ask Your PDF**: Upload a PDF and seamlessly chat with it. DocGenius extracts the text, generates embeddings, and uses context-aware retrieval to answer your questions accurately based *only* on the document's content.
- **AI Content Generator**: Generate generic content or brainstorm ideas using Google's Gemini models directly within the app.
- **Clean UI**: A responsive, mobile-friendly interface built with a minimalistic white and custom brown theme.

## Architecture

The project is split into two main directories:

### Frontend
- **Tech Stack**: Vanilla HTML, CSS, JavaScript
- **Description**: A lightweight Single Page Application (SPA). It uses native `fetch` requests to communicate with the backend. 
- **Files**:
  - `index.html`: The main structural layout of the application.
  - `style.css`: The styling file containing theme tokens and responsive layouts.
  - `app.js`: Handles DOM manipulation, event listeners, chat history, and UI state.
  - `config.js`: Contains API configuration and URLs.

### Backend
- **Tech Stack**: Python (FastAPI/Flask)
- **AI Integration**: Google Gemini (`gemini-1.5-flash` for chat, `text-embedding-004` for vector embeddings).
- **Description**: Handles PDF text extraction, document chunking, embedding generation, vector similarity search, and AI prompt engineering.
- **Files**:
  - `main.py`: The core backend logic and API endpoints.
  - `requirements.txt`: Python dependencies.
  - `.env`: Environment variables (e.g., `GEMINI_API`).

## Setup and Installation

### Prerequisites
- Python 3.9+
- A Google Gemini API Key

### Backend Setup
1. Navigate to the `backend` directory:
   ```bash
   cd backend
   ```
2. Install the required dependencies:
   ```bash
   pip install -r requirements.txt
   ```
3. Set up your environment variables:
   - Create a `.env` file in the `backend` directory.
   - Add your Gemini API key:
     ```
     GEMINI_API=your_api_key_here
     ```
4. Run the backend server:
   ```bash
   python main.py
   ```
   *(Or the equivalent command depending on the framework used, e.g., `uvicorn main:app --reload`)*

### Frontend Setup
1. Open the `frontend/config.js` file and ensure the `API_BASE_URL` points to your backend server (e.g., `http://localhost:8000`).
2. Serve the frontend:
   You can use a simple HTTP server or an extension like VS Code Live Server.
   ```bash
   cd frontend
   npx serve .
   ```
3. Open the application in your browser.

## Deployment

The application is configured to be deployed on platforms like Netlify (Frontend) and Render/Heroku (Backend). 
- **Frontend**: A `netlify.toml` file is provided for seamless Netlify deployments.
- **Backend**: `runtime.txt` and `render.yaml` are provided for Render deployments. Ensure you set your `GEMINI_API` environment variable in your hosting platform.
