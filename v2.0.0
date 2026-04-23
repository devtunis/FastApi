# copy right MIT
# powerd by power ::
import uvicorn
import sqlite3
from fastapi import FastAPI
from pydantic import BaseModel
from fastapi.middleware.cors import CORSMiddleware
from datetime import datetime

def get_today_date():
    return datetime.now().date()

# ---------------- DB ----------------
conn = sqlite3.connect("projectpfa.db", check_same_thread=False)
cursor = conn.cursor()

app = FastAPI()
origins = [
    "http://localhost:3000",
    "http://127.0.0.1:8000"
]

app.add_middleware(
    CORSMiddleware,
    allow_origins=origins,
    allow_credentials=True,
    allow_methods=["*"],
    allow_headers=["*"],
)
#delte table
#cursor.execute("""
 #   drop table register
  #  """)
#cursor.execute("""
  #  drop table groupStudy
  #  """)

#cursor.execute("""
   # drop table members
   # """)



cursor.execute("""
    CREATE TABLE IF NOT EXISTS register (
        uuid INTEGER PRIMARY KEY,
        name TEXT,
        email TEXT,
        password TEXT,
        idgroup INTEGER
    )
    """)

cursor.execute("""
    CREATE TABLE IF NOT EXISTS groupStudy (
        identify TEXT, 
        idgroupSelf INTEGER,
        namegroup TEXT,
        topic TEXT,
        datecreate TEXT,
        contentgroup TEXT,
        PRIMARY KEY (idgroupSelf, datecreate,identify),
        FOREIGN KEY (idgroupSelf) REFERENCES register(idgroup)
    )
    """)

cursor.execute("""
    CREATE TABLE IF NOT EXISTS members (
         
        userid INTEGER,
        groupid INTEGER,
        identify TEXT,
        Date TEXT,
       
        PRIMARY KEY (userid, groupid,Date),
        FOREIGN KEY (userid) REFERENCES register(id)
        FOREIGN KEY (groupid) REFERENCES groupStudy(idgroupSelf)
        FOREIGN KEY (identify) REFERENCES groupStudy(identify)
    )
    """)


cursor.execute("""
    CREATE TABLE IF NOT EXISTS tasks (
        taskId INTEGER PRIMARY KEY AUTOINCREMENT,
        
        identify TEXT,
        Date TEXT,
        content TEXT,
        nametask TEXT,
        FOREIGN KEY (identify) REFERENCES groupStudy(id)
    )
""")


conn.commit()

# ---------------- APP ----------------


# ---------------- MODEL ----------------
class User(BaseModel):
    uuid :int  
    name: str
    email: str | None = None
    password: str
    #idgroup: int
class Group(BaseModel):
    idgroup : int
    namegroup : str
    topic : str
    datecreate :str
    contentgroup : str
    identify : str
class Login(BaseModel):
    email : str
    password:str
    
class BodyGroup (BaseModel):
    userid: int 
    groupid: int  
    identify :str 
    Date :str
    
class TASK (BaseModel) :
    identify  : str  
    Date: str  
    content : str
    nametask : str
# ---------------- FUNCTION ----------------
def adduser(uuid,name, email, password):
    cursor.execute("""
        INSERT INTO register (uuid,name, email, password, idgroup)
        VALUES (?, ?, ?, ?,?)
    """, (uuid,name, email, password, uuid))

    conn.commit()
    
def addgroup(idgroup,namegroup,topic,datecreate,contentgroup,identify):
    cursor.execute("""
        INSERT INTO groupStudy (idgroupSelf, namegroup, topic, datecreate,contentgroup,identify)
        VALUES (?, ?, ?,?,?,?)
    """, (idgroup, namegroup, topic, datecreate,contentgroup,identify))    

# ---------------- ROUTE ----------------
@app.post("/register")
def register(user: User):
 
        
    adduser(
        user.uuid,
        user.name,
        user.email,
        user.password,
        
    )

    return {"message": "success"}

@app.post("/Creategroup")
def Createroup(group : Group):

    addgroup(group.idgroup,group.namegroup,group.topic,group.datecreate,group.contentgroup,group.identify)
    conn.commit()
    return {"message": "create success"}

@app.get("/fetchdeb")
def fetchDb():
   
    cursor.execute("SELECT * FROM register")
    rows3 = cursor.fetchall()
    
    cursor.execute("SELECT * FROM groupStudy")
    rows2 = cursor.fetchall()
    
    cursor.execute("SELECT groupid,userid FROM members")
    rows1 = cursor.fetchall()
    
    
    return {"register":rows3 ,"groupStudy":rows2,"memebers":rows1}

@app.get("/getsizerepo/{identify}")
def sizerepo(identify:str):
    cursor.execute("""
        SELECT name,userid ,Date
        FROM members AS m ,register AS r 
        
        WHERE m.userid =r.uuid  AND groupid = ?
    """, (identify,))
   
    
    columns = [col[0] for col in cursor.description]
    rows = cursor.fetchall()

    if not rows:
        return {"message": "No groups found for this user"}

    groups = [dict(zip(columns, row)) for row in rows]




    return {"mebmerId":groups}
   
   

@app.post("/login")
def root_login(l: Login):
    print(l.email, l.password)

    # 1. Check user in users table (NOT register)
    cursor.execute("""
        SELECT * FROM register
        WHERE email = ? AND password = ?
    """, (l.email, l.password))

    user = cursor.fetchone()

    if not user:
        return {"message": "Invalid email or password"}

 

 
 
    data = {
        "uuid" : user[0],
        "name" : user[1],
        "email" : user[2],
        #"password" : user[3],
        "groupid":user[4]
        }

    return   data 
      
   

 
 
@app.get("/getmygroups/{id}")
def get_my_groups(id: int):
    cursor.execute("""
        SELECT idgroupSelf, identify, topic, datecreate, contentgroup,namegroup
        FROM register
        JOIN groupStudy
        ON register.idgroup = groupStudy.idgroupSelf
        WHERE register.id = ?
    """, (id,))

    columns = [col[0] for col in cursor.description]
    rows = cursor.fetchall()

    if not rows:
        return {"message": "No groups found for this user"}

    groups = [dict(zip(columns, row)) for row in rows]

    return {"groups": groups}

@app.get("/groups")
def get_my_groups():
    cursor.execute("""
        SELECT idgroupSelf, identify, topic, datecreate, contentgroup, namegroup
        FROM register
        JOIN groupStudy
        ON register.idgroup = groupStudy.idgroupSelf
    """)

    columns = [col[0] for col in cursor.description]
    rows = cursor.fetchall()

    if not rows:
        return {"message": "No groups found"}

    groups = [dict(zip(columns, row)) for row in rows]

    return {"groups": groups}


def addMemberUsr(userId,groupId,identify,Date):
    cursor.execute("""
        INSERT INTO members (userid, groupid,identify, Date)
        VALUES (?, ?, ?,?)
    """, (userId, groupId,identify, Date))

@app.post("/addMember")
def addMember(p :BodyGroup):

    addMemberUsr(p.userid,p.groupid,p.identify,p.Date) 
    return  "succes"

@app.get("/followGroups")
def followgroup():
    cursor.execute(""" SELECT * FROM members""")

    columns = [col[0] for col in cursor.description]
    rows = cursor.fetchall()

    if not rows:
        return {"message": "No groups found"}

    groups = [dict(zip(columns, row)) for row in rows]




    return groups
# count mebmer of group like do count() inside request
@app.get("/getfollwgroups/{id1}")
def getfollowgroups(id1: int):
    cursor.execute("""
        SELECT g.idgroupSelf,g.identify , g.topic ,g.contentgroup,g.datecreate ,g.namegroup
        FROM members AS m
        JOIN groupStudy AS g
        ON m.groupid = g.idgroupSelf
        WHERE m.userid = ? AND g.identify = m.identify 
    """, (id1,))
    
    rows = cursor.fetchall()
    if not rows:
        return "not found"
    
    result = []
    for row in rows:
        result.append({
            "idgroup": row[0],
            "identify": row[1],
            "topic": row[2],
            "contentgroup": row[3],
            "datecreate":row[4],
            "name":row[5]
        })

    return result
            


@app.get("/users")
def fetchusers():
    cursor.execute("SELECT * FROM register")
  
    columns = [col[0] for col in cursor.description]
    rows = cursor.fetchall()

    if not rows:
        return {"message": "No groups found for this user"}

    groups = [dict(zip(columns, row)) for row in rows]




    return {"users":groups}
   
@app.get("/mygroupsfollow/{myID}")
def seeMyGroup(myID: int):
    cursor.execute("""
        SELECT *
        FROM members AS m
        WHERE m.userid = ?
    """, (myID,))

    groups = cursor.fetchall()
    if(len(groups)==0):
        return {"you dont follow groups"}
    return {"groups": groups}
    
    
@app.get("/repo/{idrepo}")
def get_repo(idrepo: str):
    cursor.execute("""
        SELECT *
        FROM groupStudy  
        WHERE  identify = ?
    """, (idrepo,))

    rows = cursor.fetchall()

    if not rows:
        raise HTTPException(status_code=404, detail="Repository not found")

    columns = [col[0] for col in cursor.description]
    groups = [dict(zip(columns, row)) for row in rows]

    return {"repo": groups}

@app.get("/session/{idSession}")
def root__session(idSession:str):
    
     cursor.execute("""SELECT * FROM groupStudy WHERE identify = ?   """, (idSession,))
     user = cursor.fetchone()
  
     if not user:
         return {"message": "error in repo"}
 
     return  {
         "identify": user[0],
         "groupid ": user[1],
         "namegroup":user[2],
         "topic" : user[3],
         "datecreate":user[4],
         "content":user[5]
         
         }


def add__pretty_task(identify,Date,content,nametask):
    cursor.execute("""
        INSERT INTO tasks (identify, Date,content, nametask)
        VALUES (?, ?, ?,?)
    """, (identify, Date,content, nametask))


   


@app.post("/tasks")
def root__identify(ts : TASK):
    add__pretty_task(ts.identify,ts.Date,ts.content,ts.nametask)
    conn.commit()
    return ts
    






# ---------------- RUN ----------------
if __name__ == "__main__":
    uvicorn.run(app, host="127.0.0.1", port=8000)
