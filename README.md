# Trabalho de mineração de repositórios <h1>

Integrantes do grupo:
    - Brendow Paolillo Castro Isidoro
    - Gabriel Hamilton Apolinario
    - Guilherme Zago Canesin
    - Isabela de Almeida Gantzel

### Perceval <h3>
Inicialmente utilizamos o Perceval para obtermos o primeiro json (coleta_zulip.json) com as requisições do github do Zulip. 

### Filtragem dos dados <h3>
Esse Json possuia muitas informações que para nossa análise de repositório não tinham necessidade, pois não obteríamos nenhuma informação útil para nossa mineração, então através do excel diminiu-se o arquivo para outro json (sortedByUsers.json) afim de fazer uma filtragem dos dados. Nesse segundo json, apenas foi deixado as informações do usuário que finalizou o pullrequest e as informação do seu respectivo pullrequest finalizado, um por um caso tivesse mais de um pullrequest finalizado por aquele usuário. A seguir uma demonstração de como ficou o json com os dados filtrados:

ˋˋˋjson
"1":{
    "USER": "007",
    "COLETA": 202,
    "TITULO": "Generic cache references",
    "BODY": "Related to #378, switch all non-specific cache references to generic `remote_cache` instead of `memcached`.\n\nWould have just switched `\\w*memcached\\w*` to `\\w*cache\\w*`, but there are a few functions / variables where they would overlap and might cause problems.\n\nThere are still a ton of references to memcached throughout the code, but **hopefully** those are all actual memcached references, and some will stay even if the backing store is changed (e.g. [zerver/lib/cache.py `cache`](https://github.com/zulip/zulip/blob/master/zerver/lib/cache.py#L218)).",
    "PULLNUMBER": 608,
    "EMAIL": "",
    "NOME": "Ryan Moore"
 ˋˋˋ

### Agrupar os pullrequests por usuário <h3>
Em nossa filtragem anterior pegamos em nosso json alguns dados necessários para nossa mineração, mas os dados ainda estavam separados por pullrequest de um usuário. Então para melhor visualização separamos o json por usuário, este por vez contendo todos os seus respectivos pullrequest assim criando outro json(filtrados.json). Para obter esse json fizemos o seguinte trecho de código:

ˋˋˋpython
x = []
media = 0
not_exists = False  

with open('teste.json') as json_file:
    y = json.load(json_file)
    x.append({ "USER":y['1']["USER"], "PULLREQUESTS": [], "TITULO": [], "BODY": [] })
    
    for j in range(1,len(y)+1):
        for i in range (len(x)):
            if(x[i]["USER"] != y[str(j)]["USER"]):
                not_exists = True
            else:
                not_exists = False
        if not_exists == True:
            x.append({"USER":y[str(j)]["USER"], "PULLREQUESTS": [], "TITULO": [], "BODY": [] })
    
    for j in range(1,len(y)+1):
        for i in range (len(x)):
            if(x[i]["USER"] == y[str(j)]["USER"]):
                if y[str(j)]["PULLNUMBER"] not in x[i]["PULLREQUESTS"]:
                    x[i]["PULLREQUESTS"].append(y[str(j)]["PULLNUMBER"])
                    x[i]["TITULO"].append(y[str(j)]["TITULO"])
                    x[i]["BODY"].append(y[str(j)]["BODY"])
    
    with open("filtrados.json", 'w', encoding='utf-8') as f:
        #json.dump(x, f, ensure_ascii=False, indent=4)
 ˋˋˋ

### filtrados.json <h3>

ˋˋˋjson
{
        "USER": "1Niels",
        "PULLREQUESTS": [
            2609,
            2630,
            2714
        ],
        "TITULO": [
            "docs: Add user guide for Edit Your Profile",
            "docs: Add user guide for emoji",
            "docs: Add user guide for change stream color"
        ],
        "BODY": [
            "Hey, I added an Edit Your Profile guide to index and as edit-profile.md, furthermore uploaded two screenshots for the guide. This was part of task type A for creating documentation for the **Edit Your Profile** feature. Thanks!",
            "Hey, I added new page with a guide for using emoji in messages and removed the section about emoji from the index. Furthermore I fixed the links for:\n\n- Create a stream\n- Streams and private messages\n- signing out\n- message formatting\n\nThanks!",
            "Hey, I added a new page to the documentation on changing stream colors. I'd appreciate any comments or suggestions. Thanks!"
        ]
    },
ˋˋˋ

### Média de pullrequests <h3>
Para obtermos um melhor resultado optamos por excluir os dados dos usuários com poucos pullrequets, para isso fizemos uma média dos pullrequests e depois excluimos os usuário com menos pullrequests que a média e por fim criamos o json final(finalResults.json). A seguir veja o trecho de código feito para obter o json:

ˋˋˋpython
    qtd_pr = len(y)
    qtd_users = len(x)
    media = qtd_pr/qtd_users
    print (media)
    ind_to_remove = []   

    for i in range (len(x)):
        if  len(x[i]["PULLREQUESTS"]) < media:
            ind_to_remove.append(i) 

    # print (ind_to_remove)
    for i in ind_to_remove[::-1]:
        # print (i)
        x.pop(i)
    for i in range (len(x)):
        print (len(x[i]["PULLREQUESTS"]))
        soma = soma + len(x[i]["PULLREQUESTS"])
    print (soma)
    print (len(x))

    with open("media.json", 'w', encoding='utf-8') as f:
        json.dump(x, f, ensure_ascii=False, indent=4)
 ˋˋˋ

 ### Classificação dos usuários <h3>
 Para a classificação das áreas de atuação dos usuário foi feito manualmente o processo de classificar os pullrequests, avaliando o nome, corpo do pullrequest e arquivos de commit, dando no total de 996 e dentre eles 36 usuários, resultando no arquivo classifiedResults.json. Para a classificação deixamos as opções de ser da área das seguintes atuações: full-stack, back-end, front-end, desktop, mobile, DevOps, Data scientist, Documentation. Veja um exemplo a seguir:

ˋˋˋjson
{
    "USER": "blablacio",
    "PULLREQUESTS": [
      420,
      419,
      434,
      547,
      542,
      574,
      559,
      543,
      767,
      544,
      451
    ],
    "TITULO": [
      "Changing admin UI to utilize tabs",
      "Adding OpenBSD compatibility",
      "Fixing performance issues with user presence list",
      "Fixing regression in saving organization settings on administration page",
      "Adding major Hubot integrations to docs",
      "Fix a problem with Travis builds",
      "Fix an OpenBSD grep incompatibility",
      "Custom realm emoji",
      "Allow HTML tag brackets inside tag attributes",
      "Custom realm filters",
      "Integrating two-factor authentication"
    ],
    "BODY": [
      "Adding tabs to admin UI.",
      "There are some changes in scripts that I made sure don't break Linux instances, but needs to be tested. Other notable changes that could possibly cause problems are changes in Node and Python requirements.",
      "Fixing issue #262, now user list is rendered only partially when user logs in.",
      "Saving the organization settings form in the administration does not work due to a trivial form name mismatch caused by following revisions: 472898cfc632c771a9fcf239f22d7e0437468b17 and 58aba59e36c8fb6e93ec7d95f6a1e54c5328ccab.",
      "I've added some of the notable Hubot integrations to the /integrations page, so users are aware of them.",
      "Travis builds seem to have been failing all day, it seems due to a redirect when downloading PhantomJS.",
      "`grep` in OpenBSD doesn't have the `--files-with-matches` switch, so we have to use `ggrep` instead.",
      "Adding some functionality to allow for managing custom realm emoji from admin.",
      "When we have a tag like this, the HTML linter is getting confused:\n`<input type=\"text\" id=\"filter_pattern\" name=\"pattern\" placeholder=\"#(?P<id>[0-9]+)\" />`\n\nThis is a small and probably naive fix for the problem at hand: double quotes are counted while scanning the template and if an uneven number of quotes has been recorded before finding a closing bracket \">\", then we just continue the search.",
      "Custom realm filters can now be managed through admin. For example, you can now say that `#([a-z]+)-([0-9]+)` should be expanded to `https://jira.zulip.com/%(pattern)%`, so when you send a message that matches the pattern like `#zul-385`, it gets auto linkified to `https://jira.zulip.com/zul-385`.\n\nI'm still investigating a problem with some patterns that crash the Markdown parser.",
      "Adding basing integration of two-factor authentication to Zulip."
    ],
    "ROLES": [
      "front-end",
      "desktop",
      "front-end",
      "front-end",
      "front-end",
      "DevOps",
      "DevOps",
      "front-end",
      "DevOps",
      "DevOps",
      "DevOps"
    ]
  },
ˋˋˋ
### Conclusão <h3>

Para finalizar, o arquivo mined_results.ipynb, demonstra a analise dos dados, avaliando a quantidade de pullrequests dentro de cada categoria e a distribuição de tipos de pullrequests para cada programador.