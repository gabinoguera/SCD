# Backend Architecture for RAG Chatbox

**Agent:** backend-developer (a6bebde)
**Date:** 2025-12-16

## Overview
Comprehensive backend implementation plan for the RAG chatbox feature following hexagonal architecture patterns established in the codebase.

## 1. Domain Layer

### 1.1 ChatMessage Entity
**File:** `backend/src/domain/entities/chat_message.py`

```python
from dataclasses import dataclass, field
from datetime import datetime
from enum import Enum
from typing import Optional, List

class MessageRole(Enum):
    USER = "user"
    ASSISTANT = "assistant"
    SYSTEM = "system"

@dataclass
class SourceReference:
    """Reference to a news article used in RAG response"""
    news_id: str
    title: str
    link: str
    relevance_score: float = 0.0

    def __post_init__(self):
        if not self.news_id:
            raise ValueError("news_id cannot be empty")
        if not self.title:
            raise ValueError("title cannot be empty")
        if not self.link:
            raise ValueError("link cannot be empty")
        if not 0.0 <= self.relevance_score <= 1.0:
            raise ValueError("relevance_score must be between 0.0 and 1.0")

@dataclass
class ChatMessage:
    session_id: str
    role: MessageRole
    content: str
    created_at: datetime = field(default_factory=datetime.utcnow)
    id: Optional[str] = None
    source_references: List[SourceReference] = field(default_factory=list)

    def __post_init__(self):
        if not self.session_id:
            raise ValueError("session_id cannot be empty")
        if not self.content or not self.content.strip():
            raise ValueError("content cannot be empty")
        if len(self.content) > 10000:
            raise ValueError("content too long (max 10000 characters)")

        # Validate source_references
        if self.role == MessageRole.USER and self.source_references:
            raise ValueError("User messages cannot have source references")
```

### 1.2 ChatSession Entity
**File:** `backend/src/domain/entities/chat_session.py`

```python
from dataclasses import dataclass, field
from datetime import datetime
from typing import Optional

@dataclass
class ChatSession:
    user_id: str
    title: str = "New Chat"
    created_at: datetime = field(default_factory=datetime.utcnow)
    updated_at: datetime = field(default_factory=datetime.utcnow)
    id: Optional[str] = None

    def __post_init__(self):
        if not self.user_id:
            raise ValueError("user_id cannot be empty")
        if not self.title or not self.title.strip():
            raise ValueError("title cannot be empty")
        if len(self.title) > 200:
            raise ValueError("title too long (max 200 characters)")

    def update_title(self, new_title: str) -> None:
        """Update session title with validation"""
        if not new_title or not new_title.strip():
            raise ValueError("title cannot be empty")
        if len(new_title) > 200:
            raise ValueError("title too long (max 200 characters)")
        self.title = new_title.strip()
        self.updated_at = datetime.utcnow()
```

### 1.3 Domain Exceptions
**File:** `backend/src/domain/exceptions/chat_exceptions.py`

```python
class ChatException(Exception):
    """Base exception for chat domain"""
    pass

class ChatSessionNotFoundException(ChatException):
    """Raised when chat session is not found"""
    def __init__(self, session_id: str):
        self.session_id = session_id
        super().__init__(f"Chat session not found: {session_id}")

class ChatMessageNotFoundException(ChatException):
    """Raised when chat message is not found"""
    def __init__(self, message_id: str):
        self.message_id = message_id
        super().__init__(f"Chat message not found: {message_id}")

class UnauthorizedChatAccessException(ChatException):
    """Raised when user tries to access chat they don't own"""
    def __init__(self, user_id: str, session_id: str):
        self.user_id = user_id
        self.session_id = session_id
        super().__init__(f"User {user_id} not authorized to access session {session_id}")

class InvalidChatMessageException(ChatException):
    """Raised when chat message validation fails"""
    pass

class VectorStoreException(ChatException):
    """Raised when vector store operations fail"""
    pass
```

## 2. Application Layer - Ports

### 2.1 ChatRepository Port
**File:** `backend/src/application/ports/chat_repository.py`

```python
from abc import ABC, abstractmethod
from typing import List, Optional
from datetime import datetime
from src.domain.entities.chat_session import ChatSession
from src.domain.entities.chat_message import ChatMessage

class ChatRepository(ABC):
    """Abstract interface for chat data operations"""

    @abstractmethod
    async def create_session(self, session: ChatSession) -> ChatSession:
        """Create a new chat session"""
        pass

    @abstractmethod
    async def get_session(self, session_id: str) -> Optional[ChatSession]:
        """Get session by ID"""
        pass

    @abstractmethod
    async def list_sessions(
        self,
        user_id: str,
        limit: int = 50,
        offset: int = 0
    ) -> List[ChatSession]:
        """List user's sessions ordered by updated_at DESC"""
        pass

    @abstractmethod
    async def update_session(self, session: ChatSession) -> ChatSession:
        """Update session (e.g., title, updated_at)"""
        pass

    @abstractmethod
    async def delete_session(self, session_id: str) -> bool:
        """Delete session and all its messages"""
        pass

    @abstractmethod
    async def create_message(self, message: ChatMessage) -> ChatMessage:
        """Create a new message in a session"""
        pass

    @abstractmethod
    async def get_messages(
        self,
        session_id: str,
        limit: int = 100,
        offset: int = 0
    ) -> List[ChatMessage]:
        """Get messages for a session ordered by created_at ASC"""
        pass

    @abstractmethod
    async def count_sessions(self, user_id: str) -> int:
        """Count total sessions for a user"""
        pass
```

### 2.2 VectorStore Port
**File:** `backend/src/application/ports/vector_store.py`

```python
from abc import ABC, abstractmethod
from typing import List, Dict, Any, Optional
from dataclasses import dataclass

@dataclass
class Document:
    """Document to be indexed in vector store"""
    id: str
    text: str
    metadata: Dict[str, Any]

@dataclass
class SearchResult:
    """Result from vector similarity search"""
    id: str
    text: str
    metadata: Dict[str, Any]
    distance: float
    relevance_score: float  # Normalized 0-1

class VectorStore(ABC):
    """Abstract interface for vector database operations"""

    @abstractmethod
    async def add_documents(self, documents: List[Document]) -> int:
        """Add documents to the vector store. Returns count of added documents."""
        pass

    @abstractmethod
    async def search(
        self,
        query: str,
        top_k: int = 5,
        filters: Optional[Dict[str, Any]] = None
    ) -> List[SearchResult]:
        """
        Search for similar documents.

        Args:
            query: Search query text
            top_k: Number of results to return
            filters: Metadata filters (e.g., {"is_public": True})

        Returns:
            List of search results ordered by relevance
        """
        pass

    @abstractmethod
    async def delete_by_id(self, document_id: str) -> bool:
        """Delete a document by ID"""
        pass

    @abstractmethod
    async def get_collection_stats(self) -> Dict[str, Any]:
        """Get statistics about the collection (count, etc.)"""
        pass

    @abstractmethod
    async def reset_collection(self) -> None:
        """Delete all documents (use for reindexing)"""
        pass
```

## 3. Application Layer - Use Cases

### 3.1 Create Session Use Case
**File:** `backend/src/application/use_cases/chat/create_session.py`

```python
from src.application.ports.chat_repository import ChatRepository
from src.domain.entities.chat_session import ChatSession

class CreateSessionUseCase:
    def __init__(self, chat_repository: ChatRepository):
        self._chat_repository = chat_repository

    async def execute(self, user_id: str, title: str = "New Chat") -> ChatSession:
        """Create a new chat session"""
        session = ChatSession(
            user_id=user_id,
            title=title
        )
        return await self._chat_repository.create_session(session)
```

### 3.2 Get Session Use Case
**File:** `backend/src/application/use_cases/chat/get_session.py`

```python
from src.application.ports.chat_repository import ChatRepository
from src.domain.entities.chat_session import ChatSession
from src.domain.exceptions.chat_exceptions import (
    ChatSessionNotFoundException,
    UnauthorizedChatAccessException
)

class GetSessionUseCase:
    def __init__(self, chat_repository: ChatRepository):
        self._chat_repository = chat_repository

    async def execute(self, session_id: str, user_id: str) -> ChatSession:
        """Get session with authorization check"""
        session = await self._chat_repository.get_session(session_id)

        if not session:
            raise ChatSessionNotFoundException(session_id)

        if session.user_id != user_id:
            raise UnauthorizedChatAccessException(user_id, session_id)

        return session
```

### 3.3 List Sessions Use Case
**File:** `backend/src/application/use_cases/chat/list_sessions.py`

```python
from typing import List
from src.application.ports.chat_repository import ChatRepository
from src.domain.entities.chat_session import ChatSession

class ListSessionsUseCase:
    def __init__(self, chat_repository: ChatRepository):
        self._chat_repository = chat_repository

    async def execute(
        self,
        user_id: str,
        limit: int = 50,
        offset: int = 0
    ) -> List[ChatSession]:
        """List user's chat sessions"""
        return await self._chat_repository.list_sessions(user_id, limit, offset)
```

### 3.4 Delete Session Use Case
**File:** `backend/src/application/use_cases/chat/delete_session.py`

```python
from src.application.ports.chat_repository import ChatRepository
from src.domain.exceptions.chat_exceptions import (
    ChatSessionNotFoundException,
    UnauthorizedChatAccessException
)

class DeleteSessionUseCase:
    def __init__(self, chat_repository: ChatRepository):
        self._chat_repository = chat_repository

    async def execute(self, session_id: str, user_id: str) -> bool:
        """Delete session with authorization check"""
        session = await self._chat_repository.get_session(session_id)

        if not session:
            raise ChatSessionNotFoundException(session_id)

        if session.user_id != user_id:
            raise UnauthorizedChatAccessException(user_id, session_id)

        return await self._chat_repository.delete_session(session_id)
```

### 3.5 Send Message Use Case
**File:** `backend/src/application/use_cases/chat/send_message.py`

```python
from datetime import datetime
from src.application.ports.chat_repository import ChatRepository
from src.domain.entities.chat_message import ChatMessage, MessageRole
from src.domain.exceptions.chat_exceptions import (
    ChatSessionNotFoundException,
    UnauthorizedChatAccessException
)

class SendMessageUseCase:
    def __init__(self, chat_repository: ChatRepository):
        self._chat_repository = chat_repository

    async def execute(
        self,
        session_id: str,
        user_id: str,
        content: str
    ) -> ChatMessage:
        """
        Send a user message to a chat session.
        This only creates the user message - RAG query is separate.
        """
        # Verify session exists and user owns it
        session = await self._chat_repository.get_session(session_id)
        if not session:
            raise ChatSessionNotFoundException(session_id)
        if session.user_id != user_id:
            raise UnauthorizedChatAccessException(user_id, session_id)

        # Create user message
        message = ChatMessage(
            session_id=session_id,
            role=MessageRole.USER,
            content=content
        )

        # Update session timestamp
        session.updated_at = datetime.utcnow()
        await self._chat_repository.update_session(session)

        return await self._chat_repository.create_message(message)
```

### 3.6 Get History Use Case
**File:** `backend/src/application/use_cases/chat/get_history.py`

```python
from typing import List
from src.application.ports.chat_repository import ChatRepository
from src.domain.entities.chat_message import ChatMessage
from src.domain.exceptions.chat_exceptions import (
    ChatSessionNotFoundException,
    UnauthorizedChatAccessException
)

class GetHistoryUseCase:
    def __init__(self, chat_repository: ChatRepository):
        self._chat_repository = chat_repository

    async def execute(
        self,
        session_id: str,
        user_id: str,
        limit: int = 100,
        offset: int = 0
    ) -> List[ChatMessage]:
        """Get chat history with authorization check"""
        # Verify session exists and user owns it
        session = await self._chat_repository.get_session(session_id)
        if not session:
            raise ChatSessionNotFoundException(session_id)
        if session.user_id != user_id:
            raise UnauthorizedChatAccessException(user_id, session_id)

        return await self._chat_repository.get_messages(session_id, limit, offset)
```

### 3.7 Execute RAG Query Use Case (CORE)
**File:** `backend/src/application/use_cases/chat/execute_rag_query.py`

```python
from typing import List
from datetime import datetime
from src.application.ports.chat_repository import ChatRepository
from src.application.ports.vector_store import VectorStore
from src.application.ports.news_repository import NewsRepository
from src.domain.entities.chat_message import ChatMessage, MessageRole, SourceReference
from src.domain.exceptions.chat_exceptions import (
    ChatSessionNotFoundException,
    UnauthorizedChatAccessException
)

class ExecuteRAGQueryUseCase:
    def __init__(
        self,
        chat_repository: ChatRepository,
        vector_store: VectorStore,
        news_repository: NewsRepository
    ):
        self._chat_repository = chat_repository
        self._vector_store = vector_store
        self._news_repository = news_repository

    async def execute(
        self,
        session_id: str,
        user_id: str,
        query: str
    ) -> ChatMessage:
        """
        Execute RAG query:
        1. Verify session ownership
        2. Search vector store for relevant news
        3. Filter by permissions
        4. Generate AI response with sources
        5. Save and return assistant message
        """
        # 1. Verify session
        session = await self._chat_repository.get_session(session_id)
        if not session:
            raise ChatSessionNotFoundException(session_id)
        if session.user_id != user_id:
            raise UnauthorizedChatAccessException(user_id, session_id)

        # 2. Search vector store
        search_results = await self._vector_store.search(
            query=query,
            top_k=10  # Get more, then filter
        )

        # 3. Filter by permissions (public news + user's own news)
        relevant_news = []
        for result in search_results:
            news_id = result.metadata.get("news_id")
            if not news_id:
                continue

            # Check if public or user's own
            is_public = result.metadata.get("is_public", False)
            news_user_id = result.metadata.get("user_id", "")

            if is_public or news_user_id == user_id:
                relevant_news.append(result)

        # Limit to top 5 after filtering
        relevant_news = relevant_news[:5]

        # 4. Generate AI response (placeholder - integrate with pydantic-ai)
        context = self._build_context(relevant_news)
        ai_response = await self._generate_response(query, context)

        # 5. Build source references
        source_refs = [
            SourceReference(
                news_id=result.metadata["news_id"],
                title=result.metadata.get("title", "Untitled"),
                link=result.metadata.get("link", ""),
                relevance_score=result.relevance_score
            )
            for result in relevant_news
        ]

        # 6. Create assistant message
        assistant_message = ChatMessage(
            session_id=session_id,
            role=MessageRole.ASSISTANT,
            content=ai_response,
            source_references=source_refs
        )

        # Update session timestamp
        session.updated_at = datetime.utcnow()
        await self._chat_repository.update_session(session)

        return await self._chat_repository.create_message(assistant_message)

    def _build_context(self, search_results: List) -> str:
        """Build context string from search results"""
        context_parts = []
        for i, result in enumerate(search_results, 1):
            context_parts.append(
                f"[{i}] {result.metadata.get('title', 'Untitled')}\n"
                f"Source: {result.metadata.get('source', 'Unknown')}\n"
                f"Summary: {result.text}\n"
            )
        return "\n\n".join(context_parts)

    async def _generate_response(self, query: str, context: str) -> str:
        """
        Generate AI response using pydantic-ai.
        TODO: Integrate with pydantic-ai agent
        """
        # Placeholder - replace with actual pydantic-ai integration
        return f"Based on the available news articles, here's what I found about '{query}':\n\n{context}"
```

### 3.8 Index News Items Use Case
**File:** `backend/src/application/use_cases/chat/index_news_items.py`

```python
from typing import List
from src.application.ports.vector_store import VectorStore, Document
from src.application.ports.news_repository import NewsRepository

class IndexNewsItemsUseCase:
    def __init__(
        self,
        vector_store: VectorStore,
        news_repository: NewsRepository
    ):
        self._vector_store = vector_store
        self._news_repository = news_repository

    async def execute(
        self,
        batch_size: int = 100,
        force_reindex: bool = False
    ) -> int:
        """
        Index all public news items into the vector store.
        Returns the number of items indexed.
        """
        if force_reindex:
            await self._vector_store.reset_collection()

        # Get all public news
        offset = 0
        total_indexed = 0

        while True:
            news_items = await self._news_repository.get_public_news(
                limit=batch_size,
                offset=offset
            )

            if not news_items:
                break

            # Convert to documents
            documents = [
                Document(
                    id=item.id,
                    text=f"{item.title}\n\n{item.summary}",
                    metadata={
                        "news_id": item.id,
                        "title": item.title,
                        "link": item.link,
                        "source": item.source,
                        "category": item.category.value,
                        "user_id": item.user_id,
                        "is_public": item.is_public,
                        "created_at": item.created_at.isoformat() if item.created_at else None
                    }
                )
                for item in news_items
            ]

            # Add to vector store
            count = await self._vector_store.add_documents(documents)
            total_indexed += count

            offset += batch_size

        return total_indexed
```

## 4. Infrastructure Layer - MongoDB Repository

**File:** `backend/src/infrastructure/adapters/repositories/mongodb_chat_repository.py`

```python
from typing import List, Optional
from motor.motor_asyncio import AsyncIOMotorDatabase
from bson import ObjectId
from src.application.ports.chat_repository import ChatRepository
from src.domain.entities.chat_session import ChatSession
from src.domain.entities.chat_message import ChatMessage, MessageRole, SourceReference

class MongoDBChatRepository(ChatRepository):
    def __init__(self, database: AsyncIOMotorDatabase):
        self._db = database
        self._sessions = database.chat_sessions
        self._messages = database.chat_messages
        self._ensure_indexes()

    def _ensure_indexes(self):
        """Create indexes for optimal query performance"""
        # Session indexes
        self._sessions.create_index([("user_id", 1)])
        self._sessions.create_index([("created_at", -1)])
        self._sessions.create_index([("updated_at", -1)])

        # Message indexes
        self._messages.create_index([("session_id", 1), ("created_at", 1)])
        self._messages.create_index([("role", 1)])

    async def create_session(self, session: ChatSession) -> ChatSession:
        doc = {
            "user_id": session.user_id,
            "title": session.title,
            "created_at": session.created_at,
            "updated_at": session.updated_at
        }
        result = await self._sessions.insert_one(doc)
        session.id = str(result.inserted_id)
        return session

    async def get_session(self, session_id: str) -> Optional[ChatSession]:
        doc = await self._sessions.find_one({"_id": ObjectId(session_id)})
        return self._doc_to_session(doc) if doc else None

    async def list_sessions(
        self,
        user_id: str,
        limit: int = 50,
        offset: int = 0
    ) -> List[ChatSession]:
        cursor = self._sessions.find(
            {"user_id": user_id}
        ).sort("updated_at", -1).skip(offset).limit(limit)

        docs = await cursor.to_list(length=limit)
        return [self._doc_to_session(doc) for doc in docs]

    async def update_session(self, session: ChatSession) -> ChatSession:
        await self._sessions.update_one(
            {"_id": ObjectId(session.id)},
            {"$set": {
                "title": session.title,
                "updated_at": session.updated_at
            }}
        )
        return session

    async def delete_session(self, session_id: str) -> bool:
        # Delete all messages first
        await self._messages.delete_many({"session_id": session_id})
        # Delete session
        result = await self._sessions.delete_one({"_id": ObjectId(session_id)})
        return result.deleted_count > 0

    async def create_message(self, message: ChatMessage) -> ChatMessage:
        doc = {
            "session_id": message.session_id,
            "role": message.role.value,
            "content": message.content,
            "source_references": [
                {
                    "news_id": ref.news_id,
                    "title": ref.title,
                    "link": ref.link,
                    "relevance_score": ref.relevance_score
                }
                for ref in message.source_references
            ],
            "created_at": message.created_at
        }
        result = await self._messages.insert_one(doc)
        message.id = str(result.inserted_id)
        return message

    async def get_messages(
        self,
        session_id: str,
        limit: int = 100,
        offset: int = 0
    ) -> List[ChatMessage]:
        cursor = self._messages.find(
            {"session_id": session_id}
        ).sort("created_at", 1).skip(offset).limit(limit)

        docs = await cursor.to_list(length=limit)
        return [self._doc_to_message(doc) for doc in docs]

    async def count_sessions(self, user_id: str) -> int:
        return await self._sessions.count_documents({"user_id": user_id})

    def _doc_to_session(self, doc: dict) -> ChatSession:
        return ChatSession(
            id=str(doc["_id"]),
            user_id=doc["user_id"],
            title=doc["title"],
            created_at=doc["created_at"],
            updated_at=doc["updated_at"]
        )

    def _doc_to_message(self, doc: dict) -> ChatMessage:
        return ChatMessage(
            id=str(doc["_id"]),
            session_id=doc["session_id"],
            role=MessageRole(doc["role"]),
            content=doc["content"],
            source_references=[
                SourceReference(
                    news_id=ref["news_id"],
                    title=ref["title"],
                    link=ref["link"],
                    relevance_score=ref.get("relevance_score", 0.0)
                )
                for ref in doc.get("source_references", [])
            ],
            created_at=doc["created_at"]
        )
```

## Summary

This architecture provides:
- **Clean separation of concerns** following hexagonal architecture
- **Full type safety** with dataclasses and validation
- **Comprehensive error handling** with domain exceptions
- **Scalable design** with repository and vector store abstractions
- **MongoDB optimization** with proper indexing
- **RAG implementation** ready for pydantic-ai integration

**Next Steps:**
1. Implement ChromaDB vector store adapter
2. Integrate pydantic-ai for AI response generation
3. Build web layer (DTOs, mappers, routers)
4. Create indexing script
5. Write comprehensive tests
