```python
import uuid
import fastapi
from fastapi import Depends, WebSocket, WebSocketDisconnect

app = fastapi.FastAPI()


from sqlalchemy import create_engine, ForeignKey, select
from sqlalchemy.orm import sessionmaker, Session, declarative_base, DeclarativeBase, Mapped, mapped_column, relationship


engine = create_engine("sqlite:///test.db")
session = sessionmaker(
    bind=engine
)

async def get_session() :
    with session() as sess:
        yield sess
class Base(DeclarativeBase):
 ...

class User(Base):
   __tablename__ ="user"
   id:Mapped[int] = mapped_column(primary_key=True)
   username:Mapped[str]
   password:Mapped[str]
   chats:Mapped[list["Chat"]] = relationship(uselist=True, back_populates="users", secondary="users_chats")


class UserChat(Base):
   __tablename__ = "users_chats"
   user_id:Mapped[int] = mapped_column(ForeignKey("user.id"), primary_key=True)
   chat_id:Mapped[uuid.UUID] = mapped_column(ForeignKey("chat.chat_id"), primary_key=True)

class Chat(Base):
   __tablename__ = "chat"
   chat_id:Mapped[uuid.UUID] = mapped_column(unique=True, primary_key=True)
   users:Mapped[list["User"]] = relationship(uselist=True, back_populates="chats", secondary="users_chats")
   messages:Mapped[list["Message"]] = relationship(uselist=True)


class Message(Base):
    __tablename__="message"
    id:Mapped[int] = mapped_column(primary_key=True)
    message:Mapped[str]
    chat_id:Mapped[uuid.UUID]  = mapped_column(ForeignKey("chat.chat_id"))



@app.get("/create-data")
def get_data():

   Base.metadata.create_all(engine)
   with session() as ses:
    ses:Session
    user = User(username = "admin1", password ="1234")
    user2 = User(username = "admin12", password ="122334")
    user3 = User(username = "admin123", password ="1232334")
    chat1 = Chat(chat_id = uuid.uuid4())
    chat2 = Chat(chat_id = uuid.uuid4())
    chat1.users.append(user)
    chat1.users.append(user2)
    chat2.users.append(user3)
    chat2.users.append(user2)
    ses.add_all([chat1,chat2])
    ses.commit()


   return 200


@app.get("/chats")
def get_chats(username:str, session:Session = Depends(get_session)):
   user= session.scalar(select(User).where(User.username==username))
   print(user)
   return user.chats


class ConnectionManager:
    def __init__(self):
        self.channels: dict[uuid.UUID, list[WebSocket]] = {}

    async def connect(self, channel: str, websocket: WebSocket):
        await websocket.accept()
        if channel not in self.channels:
            self.channels[channel] = []
        self.channels[channel].append(websocket)

    def disconnect(self, channel: str, websocket: WebSocket):
        self.channels[channel].remove(websocket)
        if not self.channels[channel]:
            del self.channels[channel]

    async def broadcast(self, channel: str, message: str):
        if channel in self.channels:
            for connection in self.channels[channel]:
                await connection.send_text(message)

channels = ConnectionManager()

@app.websocket("/ws/chat/{chat_id}")
async def websocket_endpoint(websocket: WebSocket, chat_id:uuid.UUID, ses:Session = Depends(get_session)):
    chat = ses.scalar(select(Chat).where(Chat.chat_id == chat_id))
    await channels.connect(chat_id, websocket)
    data = []
    for message in chat.messages:
     data.append(message.message)
    print(data)

    await websocket.send_json(data  )
    print(websocket.headers)
    try:
        while True:

            data = await websocket.receive_text()
            with session() as ses:
               message = Message(message= data, chat_id = chat_id)
               ses.add(message)
               ses.commit()
            await channels.broadcast(chat_id, data)

    except WebSocketDisconnect:
        print("Client disconnected")
```