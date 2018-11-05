# Bloodhound-Cypher
BH Cypher Queries picked up from random places

## Top Ten Users with Most Local Admin Rights

```
MATCH (n:User),(m:Computer), (n)-[r:AdminTo]->(m) 
WHERE NOT n.name STARTS WITH 'ANONYMOUS LOGON' 
AND NOT n.name='' 
WITH n, count(r) as rel_count order by rel_count desc 
LIMIT 10 
MATCH p=(m)<-[r:AdminTo]-(n) 
RETURN p
```

## Top Ten Computers with Most Sessions

```
MATCH (n:User),(m:Computer), (n)<-[r:HasSession]-(m) 
WHERE NOT n.name STARTS WITH 'ANONYMOUS LOGON' 
AND NOT n.name='' WITH m, count(r) as rel_count order by rel_count desc 
LIMIT 10 
MATCH p=(m)-[r:HasSession]->(n) 
RETURN n,r,m
```

## Top Ten Users with Most Sessions

```
MATCH (n:User),(m:Computer), (n)<-[r:HasSession]-(m) 
WHERE NOT n.name STARTS WITH 'ANONYMOUS LOGON' AND NOT n.name='' WITH n, count(r) as rel_count order by rel_count desc 
LIMIT 10 
MATCH p=(m)-[r:HasSession]->(n) 
RETURN p
```

## Top Ten Computers with Most Admins

```
MATCH (n:User),(m:Computer), (n)-[r:AdminTo]->(m) 
WHERE NOT n.name STARTS WITH 'ANONYMOUS LOGON' 
AND NOT n.name='' WITH m, count(r) as rel_count order by rel_count desc 
LIMIT 10 
MATCH p=(m)<-[r:AdminTo]-(n) 
RETURN p
```

## Return a list of users who have admin rights on at least one system either explicitly or through group membership

```
MATCH (u:User)-[r:AdminTo|MemberOf*1..]->(c:Computer
RETURN u.name
```

## Return username and number of computers that username is admin for, for top N users

```
MATCH 
(U:User)-[r:MemberOf|:AdminTo*1..]->(C:Computer)
WITH
U.name as n,
COUNT(DISTINCT(C)) as c 
RETURN n,c
ORDER BY c DESC
LIMIT 5
```

## Return username and number of computers that username is admin for, for top N users

```
MATCH 
(G:Group)-[r:MemberOf|:AdminTo*1..]->(C:Computer)
WITH
G.name as n,
COUNT(DISTINCT(C)) as c 
RETURN n,c
ORDER BY c DESC
LIMIT 5
```

## Show all users that are administrator on more than one machine

```
MATCH 
(U:User)-[r:MemberOf|:AdminTo*1..]->(C:Computer)
WITH
U.name as n,
COUNT(DISTINCT(C)) as c 
WHERE c>1
RETURN n
ORDER BY c DESC
```

## Show all users that are administrative on at least one machine, ranked by the number of machines they are admin on

```
MATCH (u:User)
WITH u
OPTIONAL MATCH (u)-[r:AdminTo]->(c:Computer)
WITH u,COUNT(c) as expAdmin
OPTIONAL MATCH (u)-[r:MemberOf*1..]->(g:Group)-[r2:AdminTo]->(c:Computer)
WHERE NOT (u)-[:AdminTo]->(c)
WITH u,expAdmin,COUNT(DISTINCT(c)) as unrolledAdmin
RETURN u.name,expAdmin,unrolledAdmin,expAdmin + unrolledAdmin as totalAdmin
ORDER BY totalAdmin ASC
```

## Return cross domain 'HasSession' relationships

```
MATCH p=((S:Computer)-[r:HasSession*1]->(T:User)) 
WHERE NOT S.domain = T.domain
RETURN p
```

## Find all other Rights Domain Users shouldn't have

```
MATCH p=(m:Group)-[r:Owns|:WriteDacl|:GenericAll|:WriteOwner|:ExecuteDCOM|:GenericWrite|:AllowedToDelegate|:ForceChangePassword]->(n:Computer) 
WHERE m.name STARTS WITH ‘DOMAIN USERS’ 
RETURN p
```

## List all Kerberoastable Accounts

```
MATCH (n:User)WHERE n.hasspn=true RETURN n
```

## Show Kerberoastable high value targets

```
MATCH (n:User)-[r:MemberOf]->(g:Group) 
WHERE g.highvalue=true AND n.hasspn=true 
RETURN n, g, r
```

## List Computers where DOMAIN USERS are Local Admin

```
MATCH p=(m:Group)-[r:AdminTo]->(n:Computer) 
WHERE m.name STARTS WITH ‘DOMAIN USERS’ 
RETURN p
```

## Find Workstations where DOMAIN USERS can RDP To

```
MATCH p=(g:Group)-[:CanRDP]->(c:Computer) 
WHERE g.name STARTS WITH ‘DOMAIN USERS’ 
AND NOT c.operatingsystem CONTAINS ‘Server’ 
RETURN p
```

## Find Servers where DOMAIN USERS can RDP To

```
MATCH p=(g:Group)-[:CanRDP]->(c:Computer) 
WHERE g.name STARTS WITH ‘DOMAIN USERS’ AND c.operatingsystem CONTAINS ‘Server’ 
RETURN p
```

## ALL Path from DOMAIN USERS to High Value Targets

```
MATCH (g:Group) 
WHERE g.name STARTS WITH 'DOMAIN USERS'  
MATCH (n {highvalue:true}),p=shortestPath((g)-[r*1..]->(n)) 
RETURN p
```

## Shortest Path from DOMAIN USERS to High Value Targets

```
MATCH (g:Group),(n {highvalue:true}),p=shortestPath((g)-[r*1..]->(n)) 
WHERE g.name STARTS WITH 'DOMAIN USERS' 
RETURN p
```
