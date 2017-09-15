# GraphQL

---

### What is GraphQL?

> A set of agreements

{% fragment %}
> A query language for APIs

{% fragment %}

Note:
- similar to REST, graphql is a set of agreements

---

### What is the specifications?
- Has only 1 end-point: `/graphql` 
- Independent from data model, platform, and framework
- Answer on what is asked

Query
```
{
    order(id: 1) {
        status
    }
}
```
Response
```
{
    order: {
        status: "delivering"
    }
}
```

Note: 
- Unlike REST, graphql has only 1 end point

---

Huh?
![](theme/implementing_graphql.jpg)

---

### How we can solve the api problem at the beginning of the talk?

Query
```
{
    orders {
        status
        store: {
            name
        }
    }
}
```

Response
```
{
    orders: [
        {
            status: "delivering"
            store: {
                name: "Fair Price"
            }
        },
        ...
    ]
]
```

---

### So can GraphQL solve Jon's problems?

- Multiple round trip {% fragment %}
- Over-fetching data  {% fragment %}
- Too many custom end-points  {% fragment %}
- Slow to fulfill request from api users  {% fragment %}

Note:
- Multiple round trip: with just 1 api call, client can retrieve any related data
- Over-fetching data: this won't happend as client are always the one who specify what to return
- Too many custom end-points: just 1 end point /graphql, serve all king of thing
- Slow to fulfill api users request: no need to wait for backend, client can customize the payload however they want.

-> this lead to:
- backend: no one keep asking for end-point customization, only 1 end-point to maintain, backend is happy
- api user: no need to wait for backend to customize response, no blocker, front end is happy
- PM: feature roll out smoothly without delay, PM is happy
- customer: web is fast, app is fast, use much less data
- merchant partner: API is so flexible and easy to use, merchant's happy, merchant onboard
-> Jon is now finally happy


---

{% background green %}

## Everyone is happy
## Jon is happy! {% fragment %}

---

{% background green %}
### Jon can finally sleep well at night again
![](/theme/bed.png)


