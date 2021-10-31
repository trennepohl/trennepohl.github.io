# Kubernetes e performance de DNS - ndots


# Introdução
Hoje eu aprendi sobre uma configuração de DNS do Kubernetes que existe em vários setups, talvez exista no seu e que nem eu, você nem saiba!

## Contexto
Hoje eu aprendi sobre ndots.
ndots é um parâmetro do arquivo `/etc/resolv.conf` que configura o número mínimo de pontos ( . ) que um nome (DNS) deve ter


## Cenário

Dependendo do seu setup do kubernetes,os pods podem conter o seguinte conteúdo do resolv.conf :
```
search default.svc.cluster.local svc.cluster.local cluster.local
nameserver 10.96.0.10
options ndots:5
```

Reparem que o ndots está configurado para 5
Agora imagine que você tenha um serviço (apiv1/Service) chamado pagamentos e que hipoteticamente existam outros pod nesse cluster que utilizem o DNS interno para enviar requests ao serviço de pagamento.

A url da request seria algo do tipo: 
`pagamentos-svc.pagamentos.svc.cluster.local:3000` (4 pontos)

O resolvedor de DNS vai dizer, pera lá 4 é menor que 5 e vai buscar esse domínio em cada um dos nameservers até achar o mesmo.
```
default.svc.cluster.local
svc.cluster.local
cluster.local
```
Agora imagine que esse cluster faz 5k de requests por segundo para o serviço de pagamentos. 
Pobre do kube-dns (ou core-dns tbm) né? CPU throttling vai lá em cima e os serviços começam a ficar mais lentos por causa de DNS

A solução imediata seria adicionar um ponto ao fim da URL.<br>
`pagamentos-svc.pagamentos.svc.cluster.local.:3000` <br>
isso vai fazer com que o resolvedor de DNS não pingue de nameserver em nameserver e tente resolver exatamente esse endereço.

## Testando
Se vocẽ tem o prometheus rodando no seu cluster, você será capaz de visualizar a diferença facilmente.

Crie alguns pods em uma namespace (X) e um service em outra namespace (Y) e escreva um script simples para forçar a resolução de nome, exemplo: 

`for i in {1..5000}; do nslookup app.y.svc.cluster.local}; done`<br>
e depois<br>
`for i in {1..5000}; do nslookup app.y.svc.cluster.local.}; done`

E compare os gráficos de uso de CPU e requests no Grafana.


## Referências
https://pracucci.com/kubernetes-dns-resolution-ndots-options-and-why-it-may-affect-application-performances.html
https://man7.org/linux/man-pages/man5/resolv.conf.5.html
