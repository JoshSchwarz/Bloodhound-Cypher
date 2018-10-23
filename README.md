# Bloodhound-Cypher
BH Cypher Queries picked up from random places


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

## return cross domain 'HasSession' relationships

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
match p=(g:Group)-[:CanRDP]->(c:Computer) where g.name STARTS WITH ‘DOMAIN USERS’ AND NOT c.operatingsystem CONTAINS ‘Server’ return p
```

## Find Servers where DOMAIN USERS can RDP To

```
match p=(g:Group)-[:CanRDP]->(c:Computer) where g.name STARTS WITH ‘DOMAIN USERS’ AND c.operatingsystem CONTAINS ‘Server’ return p
```
