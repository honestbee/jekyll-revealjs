This is Jon.
Jon is a young entrepreneur, he and and a few friends start a business.
They have this break-thu idea of selling vegetable online.

Wait no more, Jon start implement his product right away.

Let's assume Jon's database schema is as simple as this.
```
Orders: status, store_id
Stores: name
```

Jon implement this back-end follow the standard REST approach. Everything is easy, no issue so far. 

Jon sleep well at night.

The next day the front-end team dive in and implement some feature using our REST api above

<The simple page that look like this, show a page of order fulfillment nested inside order>

So how should he implement this?
Step 1: retrieve a list of orders `GET /orders`
```
[
{
    id: 1,
    store_id: 2,
    status: "processing"
},
...
]
```
Step 2: for each order retrieve the related store 
`GET /store/2`
`GET /store/3`
`GET /store/4`
`GET /store/5`
`...`

and that is I have stop at store, I haven't query country, user, and timeslot information.

Impact? web is slow, mobile app is slow, mobile customer waste a ton of data, customer are angry -> Jon is sad

How should we improve this?
By loading the related information and return them in the same API call.

`GET /orders`

```
[
{
    id: 1,
    store: {
        id: 2,
        name: "Fair Price"
    },
    status: "processing"
},
...
]
```

any issue with this approach?

yes, the issue with this is for most of the time we will over fetching data that we don't event need.

Impact: queries are slower in average, web and app are unresponsive due to every request are heavy, customer waste a bit more time, a bit more data. Customers are not happy -> Jon is a little bit sad

Idea?

Solution: custom end-point

```
GET order/
GET order_with_store/
GET order_with_store_and_country/
GET order_with_store_and_country_and_timeslot/
GET order_with_store_and_country_and_timeslot_and_customer/
GET order_with_everything/
...
```

And this is the state where honestbee is at now.

<Show screenshot of so many custom seriallizers>

Everytime, a client need to use an end point a little bit different, like adding more information to the response they need to send a request to the backend team. Wait for the ticket get schedule into a sprint, wait for it to be implemented, wait for it to be reviewed, wait for it to be tested by QA, and then wait for it to be release. This whole cycle can easily take up to 1-2 week.

<show example of alex huang>
<show the misuse of seriallizer in order fulfillment controller>

From a backend developer point of view: this increase the comlexity of the API implementation, the comlexity of the API documents, more thing to maintain.

From a front-end developer point of view: this add unnecessary blocker to the workflow

From a PM point of view: this delays new feature by 1-2 week

-> Jon is sad

With all of that hassle, Jon decide it's time to find a comprehensive solution to cure all the issues.

- Multiple round trip
- Over-fetching data
- Too many custom end point
- Slow to fulfill api users request

Jon found a thing call GraphQL.

Can graphql really solve all the problem that Jon is having?
- Multiple round trip: with just 1 api call, client can retrieve any related data
- Over-fetching data: this won't happend as client are always the one who specify what to return
- Too many custom end point: just 1 end point /graphql, serve all king of thing
- Slow to fulfill api users request: no need to wait for backend, client can customize the payload however they want.

-> this lead to:
- backend: no one keep asking for end-point customization, only 1 end-point to maintain, backend is happy
- api user: no need to wait for backend to customize response, no blocker, front end is happy
- PM: feature roll out smoothly without delay, PM is happy
- customer: web is fast, app is fast, use much less data
- merchant partner: API is so flexible and easy to use, merchant's happy, merchant onboard
-> Jon is now finally happy



---

#Live demo

An order has many fulfillments

#Create new rails project
    rails new graphql-honestbee

#Add 2 model for Order and Order fulfillment
    rails g model order status:string store:belongs_to
    rails g model store name:string

- show the models in sublime
- add `has_many` relationship

- create some mock data: 
```
    Store.create name: 'Cold Storage'
    Store.create name: 'Fair Price'

    Order.create! status: :processing, store: Store.first
    Order.create! status: :pending, store: Store.second
```

#Add graphql gem

#Bootstrap graphql in your project
    rails g graphql:install
explain what what this step add to the project

- show GraphiQL, test a query on testField
- delete testField decleration and see what if the error

- Add Order Type
```
Types::OrderType = GraphQL::ObjectType.define do
  name "Order"
  field :id, types.ID
  field :status, types.String
end
```

- add order field to query
```
field :order do
    type Types::OrderType
    argument :id, !types.ID
    resolve ->(_object, arguments, _context) do
      Order.find(arguments[:id])
    end
end
```

- add store type
```
Types::StoreType = GraphQL::ObjectType.define do
  name "Store"
  field :id, types.ID
  field :name, types.String
end
```

- add store field to order type

- Analyze query result, discover there is a n+1 situation

- Decide to use batch loader
- Create a loader class
```
# Basic loader for Rails' records
class Loaders::RecordLoader < GraphQL::Batch::Loader
  def initialize(model)
    @model = model
  end

  def perform(ids)
    @model.where(id: ids).each { |record| fulfill(record.id, record) }
    ids.each { |id| fulfill(id, nil) unless fulfilled?(id) }
  end
end

```

- Add `use GraphQL::Batch` to the root schema file

- Add this to the field
```
resolve -> (obj, args, context) { Loaders::RecordLoader.for(Store).load(obj["id"]) }
```

- Show that the n+1 issue has gone

--- 

QUESTION?
