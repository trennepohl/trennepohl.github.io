# Mongodb aggregations in Golang


# Intro

Sometimes when you google for this, the answer is not quite clear and I also often forget how to do this.

### Scenario

Imagine the following dataset in a mongodb collection

``` 
{ _id: 1, cust_id: "abc1", ord_date: ISODate("2012-11-02T17:04:11.102Z"), status: "A", amount: 50 }
{ _id: 2, cust_id: "xyz1", ord_date: ISODate("2013-10-01T17:04:11.102Z"), status: "A", amount: 100 }
{ _id: 3, cust_id: "xyz1", ord_date: ISODate("2013-10-12T17:04:11.102Z"), status: "D", amount: 25 }
{ _id: 4, cust_id: "xyz1", ord_date: ISODate("2013-10-11T17:04:11.102Z"), status: "D", amount: 125 }
{ _id: 5, cust_id: "abc1", ord_date: ISODate("2013-11-12T17:04:11.102Z"), status: "A", amount: 25 }

```

&nbsp;
&nbsp;

And we want to know the total count of prodcuts by order status.


#### Group

```
type OrderStatusTotal struct {
	ID string `bson:"_id"`
	Total int `bson:"total"`
}

pipelineResult := make([]OrderStatusTotal, 0)

pipeline := make([]bson.M, 0)

groupStage := bson.M{
    "$group": bson.M{
    "_id": "$status",
    "total": bson.M{"$sum": 1},
    },
}

pipeline = append(pipeline, groupStage)

data, err := collection.Aggregate(ctx, pipeline)
if err != nil {
    log.Println(err.Error())
    fmt.Errorf("failed to execute aggregation %s", err.Error())
    return
}

err = data.All(ctx, &pipelineResult)
if err != nil {
    log.Println(err.Error())
    fmt.Errorf("failed to decode results", err.Error())
    return
}

fmt.Printf("%+v\n", pipelineResult)

}

```

#### Group And Match

```
type OrderStatusTotal struct {
	ID string `bson:"_id"`
	Total int `bson:"total"`
}

pipelineResult := make([]OrderStatusTotal, 0)

pipeline := make([]bson.M, 0)

groupStage := bson.M{
    "$group": bson.M{
    "_id": "$status",
    "total": bson.M{"$sum": 1},
    },
}

matchStage := bson.M{
    "$match": bson.M{
        "cust_id": "abc1",
    },
}

pipeline = append(pipeline, matchStage,groupStage)

data, err := collection.Aggregate(ctx, pipeline)
if err != nil {
    log.Println(err.Error())
    fmt.Errorf("failed to execute aggregation %s", err.Error())
    return
}

err = data.All(ctx, &pipelineResult)
if err != nil {
    log.Println(err.Error())
    fmt.Errorf("failed to decode results", err.Error())
    return
}

fmt.Printf("%+v\n", pipelineResult)

```

# Notes
- This is a super simple example, do not use in production

- In order to avoid many external dependencies I always create custom types for querying mongodb, e.g

```
 type DBQuery map[string]interface{}
```

- Avoid using empty interfaces, this only leads to nil pointers and type assertions

- [Mongo aggregation docs](https://docs.mongodb.com/manual/reference/method/db.collection.aggregate/)

- Feedbacks are always welcome :)


Full example can be found [here](https://gist.github.com/trennepohl/bd756c29b31398cca0c7212bb6e9961b)
