# Mongodb: Como fazer agregações/pipelines em Golang


# Introdução
Algumas vezes que procurei no google por isso a resposta nao estava exatamente clara e como eu tenho uma memória horrível 
decidi criar esse post.

### Cenário

Imagine a seguinte coleção de dados em uma collection do mongodb.

``` 
{ _id: 1, cust_id: "abc1", ord_date: ISODate("2012-11-02T17:04:11.102Z"), status: "A", amount: 50 }
{ _id: 2, cust_id: "xyz1", ord_date: ISODate("2013-10-01T17:04:11.102Z"), status: "A", amount: 100 }
{ _id: 3, cust_id: "xyz1", ord_date: ISODate("2013-10-12T17:04:11.102Z"), status: "D", amount: 25 }
{ _id: 4, cust_id: "xyz1", ord_date: ISODate("2013-10-11T17:04:11.102Z"), status: "D", amount: 125 }
{ _id: 5, cust_id: "abc1", ord_date: ISODate("2013-11-12T17:04:11.102Z"), status: "A", amount: 25 }

```

&nbsp;
&nbsp;

E queremos saber o total de produtos por estado do pedido.


#### Agrupamento/Group

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

#### Agrupamento e Filtros/Group and Match

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
### Notas
- O código deste post é um exemplo muito simples, não está pronto para produção.

- Para evitar dependencias externas eu costumo criar um type alias para as queries do mongo, exemplo:
        ```type DBQuery map[string]interface{}```

- Evite usar interfaces vazias, isso só vai te causar nil pointers e muitos type assertions

- [Documentação do mongo sobre agregações e pipelines](https://docs.mongodb.com/manual/reference/method/db.collection.aggregate/)

- Feedbacks são bem vindos :)


Exemplo completo pode ser encondato  [aqui](https://gist.github.com/trennepohl/bd756c29b31398cca0c7212bb6e9961b)
