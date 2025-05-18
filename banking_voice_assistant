from typing import List, Dict, Optional
from pathlib import Path
import os
from firecrawl import FirecrawlApp
from qdrant_client import QdrantClient
from qdrant_client.http import models
from qdrant_client.http.models import Distance, VectorParams
from fastembed import TextEmbedding
from agents import Agent, Runner
from openai import AsyncOpenAI
import tempfile
import uuid
from datetime import datetime
import time
import streamlit as st
from dotenv import load_dotenv
import asyncio

load_dotenv()

def init_session_state():
    defaults = {
        "initialized": False,
        "qdrant_url": "",
        "qdrant_api_key": "",
        "firecrawl_api_key": "",
        "openai_api_key": "",
        "doc_url": "",
        "setup_complete": False,
        "client": None,
        "embedding_model": None,
        "processor_agent": None,
        "tts_agent": None,
        "selected_voice": "nova"
    }
    
    for key, value in defaults.items():
        if key not in st.session_state:
            st.session_state[key] = value

def sidebar_config():
    with st.sidebar:
        st.title("🏦 Banking Assistant Setup")
        st.markdown("---")
        
        st.markdown("### 🔐 API Configuration")
        st.markdown("Configure your API keys to enable the banking assistant")
        
        st.session_state.qdrant_url = st.text_input(
            "Vector Database URL",
            value=st.session_state.qdrant_url,
            type="password",
            help="URL for the vector database storing banking documentation"
        )
        st.session_state.qdrant_api_key = st.text_input(
            "Vector Database API Key",
            value=st.session_state.qdrant_api_key,
            type="password"
        )
        st.session_state.firecrawl_api_key = st.text_input(
            "Document Crawler API Key",
            value=st.session_state.firecrawl_api_key,
            type="password"
        )
        st.session_state.openai_api_key = st.text_input(
            "OpenAI API Key",
            value=st.session_state.openai_api_key,
            type="password"
        )
        
        st.markdown("---")
        st.markdown("### 📚 Banking Documentation")
        st.session_state.doc_url = st.text_input(
            "Banking Documentation URL",
            value=st.session_state.doc_url,
            placeholder="https://docs.yourbank.com",
            help="URL containing your bank's documentation, policies, and procedures"
        )
        
        st.markdown("---")
        st.markdown("### 🎤 Voice Assistant Settings")
        voices = ["nova", "onyx", "echo", "fable", "shimmer"]
        st.session_state.selected_voice = st.selectbox(
            "Select Assistant Voice",
            options=voices,
            index=voices.index(st.session_state.selected_voice),
            help="Choose the voice for your banking assistant"
        )
        
        if st.button("Initialize Banking Assistant", type="primary"):
            if all([
                st.session_state.qdrant_url,
                st.session_state.qdrant_api_key,
                st.session_state.firecrawl_api_key,
                st.session_state.openai_api_key,
                st.session_state.doc_url
            ]):
                progress_placeholder = st.empty()
                with progress_placeholder.container():
                    try:
                        st.markdown("🔄 Setting up knowledge base...")
                        client, embedding_model = setup_qdrant_collection(
                            st.session_state.qdrant_url,
                            st.session_state.qdrant_api_key
                        )
                        st.session_state.client = client
                        st.session_state.embedding_model = embedding_model
                        st.markdown("✅ Knowledge base setup complete!")
                        
                        st.markdown("🔄 Learning from banking documentation...")
                        pages = crawl_documentation(
                            st.session_state.firecrawl_api_key,
                            st.session_state.doc_url
                        )
                        st.markdown(f"✅ Processed {len(pages)} banking documents!")
                        
                        store_embeddings(
                            client,
                            embedding_model,
                            pages,
                            "docs_embeddings"
                        )
                        
                        processor_agent, tts_agent = setup_agents(
                            st.session_state.openai_api_key
                        )
                        st.session_state.processor_agent = processor_agent
                        st.session_state.tts_agent = tts_agent
                        
                        st.session_state.setup_complete = True
                        st.success("✅ Banking Assistant initialized successfully!")
                        
                    except Exception as e:
                        st.error(f"Error during setup: {str(e)}")
            else:
                st.error("Please fill in all the required fields!")

def setup_qdrant_collection(qdrant_url: str, qdrant_api_key: str, collection_name: str = "docs_embeddings"):
    client = QdrantClient(url=qdrant_url, api_key=qdrant_api_key)
    embedding_model = TextEmbedding()
    test_embedding = list(embedding_model.embed(["test"]))[0]
    embedding_dim = len(test_embedding)
    
    try:
        client.create_collection(
            collection_name=collection_name,
            vectors_config=VectorParams(size=embedding_dim, distance=Distance.COSINE)
        )
    except Exception as e:
        if "already exists" not in str(e):
            raise e
    
    return client, embedding_model

def crawl_documentation(firecrawl_api_key: str, url: str, output_dir: Optional[str] = None):
    firecrawl = FirecrawlApp(api_key=firecrawl_api_key)
    pages = []
    
    if output_dir:
        os.makedirs(output_dir, exist_ok=True)
    
    response = firecrawl.crawl_url(
        url,
        params={
            'limit': 5,
            'scrapeOptions': {
                'formats': ['markdown', 'html']
            }
        }
    )
    
    while True:
        for page in response.get('data', []):
            content = page.get('markdown') or page.get('html', '')
            metadata = page.get('metadata', {})
            source_url = metadata.get('sourceURL', '')
            
            if output_dir and content:
                filename = f"{uuid.uuid4()}.md"
                filepath = os.path.join(output_dir, filename)
                with open(filepath, 'w', encoding='utf-8') as f:
                    f.write(content)
            
            pages.append({
                "content": content,
                "url": source_url,
                "metadata": {
                    "title": metadata.get('title', ''),
                    "description": metadata.get('description', ''),
                    "language": metadata.get('language', 'en'),
                    "crawl_date": datetime.now().isoformat()
                }
            })
        
        next_url = response.get('next')
        if not next_url:
            break
            
        response = firecrawl.get(next_url)
        time.sleep(1)
    
    return pages

def store_embeddings(client: QdrantClient, embedding_model: TextEmbedding, pages: List[Dict], collection_name: str):
    for page in pages:
        embedding = list(embedding_model.embed([page["content"]]))[0]
        client.upsert(
            collection_name=collection_name,
            points=[
                models.PointStruct(
                    id=str(uuid.uuid4()),
                    vector=embedding.tolist(),
                    payload={
                        "content": page["content"],
                        "url": page["url"],
                        **page["metadata"]
                    }
                )
            ]
        )

def setup_agents(openai_api_key: str):
    os.environ["OPENAI_API_KEY"] = openai_api_key
    
    processor_agent = Agent(
        name="Banking Documentation Processor",
        instructions="""You are a professional banking assistant. Your task is to:
        1. Analyze the provided banking documentation content
        2. Answer customer questions about banking services, policies, and procedures
        3. Provide clear explanations of banking terms and processes
        4. Include relevant examples when available
        5. Cite the source URLs when referencing specific content
        6. Keep responses professional, accurate, and easy to understand
        7. Format your response in a way that's easy to speak out loud
        8. Always maintain a professional and trustworthy tone""",
        model="gpt-4o"
    )

    tts_agent = Agent(
        name="Banking Voice Assistant",
        instructions="""You are a professional banking voice assistant. Your task is to:
        1. Convert banking responses into natural, professional speech
        2. Maintain proper pacing and emphasis for financial terms
        3. Handle banking terminology clearly and accurately
        4. Keep the tone professional, trustworthy, and friendly
        5. Use appropriate pauses for better comprehension
        6. Ensure the speech is clear, well-articulated, and instills confidence""",
        model="gpt-4o-mini-tts"
    )
    
    return processor_agent, tts_agent

async def process_query(
    query: str,
    client: QdrantClient,
    embedding_model: TextEmbedding,
    processor_agent: Agent,
    tts_agent: Agent,
    collection_name: str,
    openai_api_key: str
):
    try:
        query_embedding = list(embedding_model.embed([query]))[0]
        search_response = client.query_points(
            collection_name=collection_name,
            query=query_embedding.tolist(),
            limit=3,
            with_payload=True
        )
        
        search_results = search_response.points if hasattr(search_response, 'points') else []
        
        if not search_results:
            raise Exception("No relevant documents found in the vector database")
        
        context = "Based on the following banking documentation:\n\n"
        for result in search_results:
            payload = result.payload
            if not payload:
                continue
            url = payload.get('url', 'Unknown URL')
            content = payload.get('content', '')
            context += f"From {url}:\n{content}\n\n"
        
        context += f"\nUser Question: {query}\n\n"
        context += "Please provide a clear, concise answer that can be easily spoken out loud."
        
        processor_result = await Runner.run(processor_agent, context)
        processor_response = processor_result.final_output
        
        tts_result = await Runner.run(tts_agent, processor_response)
        tts_response = tts_result.final_output
        
        async_openai = AsyncOpenAI(api_key=openai_api_key)
        audio_response = await async_openai.audio.speech.create(
            model="gpt-4o-mini-tts",
            voice=st.session_state.selected_voice,
            input=processor_response,
            instructions=tts_response,
            response_format="mp3"
        )
        
        temp_dir = tempfile.gettempdir()
        audio_path = os.path.join(temp_dir, f"response_{uuid.uuid4()}.mp3")
        
        with open(audio_path, "wb") as f:
            f.write(audio_response.content)
                
        return {
            "status": "success",
            "text_response": processor_response,
            "tts_instructions": tts_response,
            "audio_path": audio_path,
            "sources": [r.payload.get("url", "Unknown URL") for r in search_results if r.payload],
            "query_details": {
                "vector_size": len(query_embedding),
                "results_found": len(search_results),
                "collection_name": collection_name
            }
        }
    
    except Exception as e:
        return {
            "status": "error",
            "error": str(e),
            "query": query
        }

def run_streamlit():
    st.set_page_config(
        page_title="Banking Voice Assistant",
        page_icon="🏦",
        layout="wide"
    )
    
    init_session_state()
    sidebar_config()
    
    st.title("🏦 Banking Voice Assistant")
    st.markdown("""
    Welcome to your AI-powered Banking Voice Assistant! This intelligent assistant can:
    - Answer questions about banking services and products
    - Explain banking policies and procedures
    - Provide information about account types and features
    - Guide you through banking processes
    - Offer voice responses for a natural conversation experience
    
    To get started:
    1. Configure your API keys in the sidebar
    2. Enter your bank's documentation URL
    3. Ask your banking-related questions below
    """)
    
    query = st.text_input(
        "What would you like to know about our banking services?",
        placeholder="e.g., How do I open a savings account? What are the current interest rates?",
        disabled=not st.session_state.setup_complete
    )
    
    if query and st.session_state.setup_complete:
        with st.status("Processing your banking query...", expanded=True) as status:
            try:
                st.markdown("🔄 Searching banking documentation and generating response...")
                result = asyncio.run(process_query(
                    query,
                    st.session_state.client,
                    st.session_state.embedding_model,
                    st.session_state.processor_agent,
                    st.session_state.tts_agent,
                    "docs_embeddings",
                    st.session_state.openai_api_key
                ))
                
                if result["status"] == "success":
                    status.update(label="✅ Response ready!", state="complete")
                    
                    st.markdown("### 📝 Banking Assistant Response:")
                    st.write(result["text_response"])
                    
                    if "audio_path" in result:
                        st.markdown(f"### 🔊 Voice Response (Voice: {st.session_state.selected_voice})")
                        st.audio(result["audio_path"], format="audio/mp3", start_time=0)
                        
                        with open(result["audio_path"], "rb") as audio_file:
                            audio_bytes = audio_file.read()
                            st.download_button(
                                label="📥 Download Voice Response",
                                data=audio_bytes,
                                file_name=f"banking_assistant_response_{st.session_state.selected_voice}.mp3",
                                mime="audio/mp3"
                            )
                    
                    st.markdown("### 📚 Information Sources:")
                    for source in result["sources"]:
                        st.markdown(f"- {source}")
                else:
                    status.update(label="❌ Error processing query", state="error")
                    st.error(f"Error: {result.get('error', 'Unable to process your banking query')}")
                    
            except Exception as e:
                status.update(label="❌ Error processing query", state="error")
                st.error(f"Error processing your banking query: {str(e)}")
    
    elif not st.session_state.setup_complete:
        st.info("👈 Please configure the Banking Assistant using the sidebar first!")

if __name__ == "__main__":
    run_streamlit()
