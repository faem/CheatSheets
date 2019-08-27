## Install
https://docs.mongodb.com/manual/tutorial/install-mongodb-on-ubuntu/

## Relation with RDBMS

| RDBMS	|MongoDB |
|--|--|
| Database |	Database |
Table |	Collection
Tuple/Row	| Document
column |	Field
Table Join |	Embedded Documents
Primary Key	| Primary Key (Default key _id provided by mongodb itself)


Database Server | Client
|--|--|
mysqld/oracle	| mysql/sqlplus	
 mongod| mongo
 
 
 ## Sample Document
 ```
 {
   _id: ObjectId(7df78ad8902c)
   title: 'MongoDB Overview', 
   description: 'MongoDB is no sql database',
   by: 'tutorials point',
   url: 'http://www.tutorialspoint.com',
   tags: ['mongodb', 'database', 'NoSQL'],
   likes: 100, 
   comments: [	
      {
         user:'user1',
         message: 'My first comment',
         dateCreated: new Date(2011,1,20,2,15),
         like: 0 
      },
      {
         user:'user2',
         message: 'My second comments',
         dateCreated: new Date(2011,1,25,7,45),
         like: 5
      }
   ]
}
```
_id is a 12 bytes hexadecimal number which assures the uniqueness of every document. 

