# Documentação de integração com o Bonsae

## Autenticação

Todas as requisições para endpoints da API (exceto DoAccess) requerem um token de autenticação enviado no corpo da requisição:

```json
{
	"token": "seu_token_api"
}
```

O token deve ser configurado na variável de ambiente `AUTH_TOKEN_API` no sistema Bonsae.

## Endpoints disponíveis

### Usuários
- `POST /api/webhook/all-users` - Listar todos os usuários
- `POST /api/webhook/create-user` - Criar novo usuário
- `POST /api/webhook/user` - Consultar usuário específico
- `POST /api/webhook/update-user` - Atualizar usuário
- `DELETE /api/webhook/delete-user` - Deletar usuário
- `GET /webhook/DoAcess` - Autenticação via hash (sem /api/)

---

### Envio de notas
O envio de notas de uma turma será feito apenas quando a turma for concluída (após o prazo final da prática estabelecido pelo professor na Bonsae) através de um job que rodará de madrugada a partir das 1 da manhã.

### Variáveis 
| variável | tipo | Descrição |
| ------ | ------ | ------ |
| class | array | Array com informações da turma|
| practices | string | Array com todas as tarefas feitas(informações dos estudantes como nota e lista de presença estão dentro do array) |

### Exemplo de requisição

```
{
	"class":[
	    "code": "TE"
	    "name":"turma teste"
	    "intern_id":1
	],
	"practices":[
	    {
    	    "name":"teste",
    	    "client":{
    	        "name":"teste",
    	        "cpf":"123.456.789-11",
    	        "telephone":"(55)99999-9999"
    	        "is_simulated":0,
    	        "email":"teste@bonsae.com",
    	        "lawsuit":"131313131"
    	    },
    	    "start_date": "2022-01-29",
    	    "start_hour":"12:00",
    	    "end_hour": "14:00"
    	    "end_date":"2022-01-30"
    	    "instruction":" teste 123",
        	"student":[
        	    {
        	        "grade":6.0,
            	    "registration_number":"12345678",
            	    "name":"aluno",
            	    "email":"aluno@teste.com",
            	    "intern_id":1,
            	    "presence":1
        	    }
        	]
	   }
	]
}
```
# Autenticação via hash

A integração via hash permite autenticação automática entre sistemas usando email e código de validação.

> **Método:** GET
> **URL:** https://instancia.bonsae.com.br/webhook/DoAcess&email={email}&code={code}

### Parâmetros da URL
| Parâmetro | Tipo | Descrição |
| ------ | ------ | ------ |
| `email*` | string | Email do usuário |
| `code*` | string | Código de verificação gerado com md5 (access_token + hash único) |

### Geração do código de autenticação
```php
// Exemplo de geração do código
$user = Auth::user();
$hash = env('HASH'); // Hash único configurado no .env
$code = md5($user->access_token . $hash);

$url = "https://instancia.bonsae.com.br/webhook/DoAcess&email={$user->email}&code={$code}";
```

### Exemplo de URL completa
```
https://instancia.bonsae.com.br/webhook/DoAcess&email=exemplo@teste.com&code=a1b2c3d4e5f6g7h8i9j0
```

### Observações
- O `access_token` deve ser armazenado no banco de dados de ambos os sistemas
- O `access_token` deve ser gerado aleatoriamente durante a criação do usuário
- O hash único deve ser o mesmo em ambos os sistemas (configurado via variável de ambiente)

# Criação de usuários via webhook

> **Método:** POST
> **URL:** https://instancia.bonsae.com.br/api/webhook/create-user
> **Autenticação:** Token obrigatório

### Tipos de usuários e seus identificadores

| Tipo de perfil | Identificador |
| ------ | ------ |
| Coordenador | 1 |
| Professor | 2 |
| Secretário | 3 |
| Aluno | 4 |
| Advogado | 5 |
| Estagiário | 6 |

### Parâmetros obrigatórios (todos os perfis)
| Variável | Tipo | Descrição |
| ------ | ------ | ------ |
| `token*` | string | Token de autenticação da API (AUTH_TOKEN_API) |
| `name*` | string | Nome do usuário |
| `email*` | string | Email do usuário (deve ser único) |
| `registration_number*` | string | Matrícula do usuário (deve ser única) |
| `access_token*` | string | Token para autenticação do usuário |
| `profile_id*` | integer | Identificador do perfil (1-6) |

### Parâmetros opcionais
| Variável | Tipo | Descrição |
| ------ | ------ | ------ |
| `password` | string | Senha do usuário (padrão: POLO@123) |
| `oab` | string | Número da OAB (para coordenador, professor ou advogado) |
| `oab_uf` | string | UF da OAB (sigla do estado, ex: "SP") |

### Exemplos de requisição

**Coordenador (com OAB e senha personalizada):**
```json
{
	"token": "seu_token_api",
	"name": "João Silva",
	"email": "coordenador@teste.com",
	"registration_number": "12345678",
	"password": "coordenador123",
	"access_token": "dsgvsd2v3f6",
	"profile_id": 1,
	"oab": "123456",
	"oab_uf": "SP"
}
```

**Professor (com senha personalizada):**
```json
{
	"token": "seu_token_api",
	"name": "Maria Santos",
	"email": "professor@teste.com",
	"registration_number": "87654321",
	"password": "professor123",
	"access_token": "abc123def456",
	"profile_id": 2
}
```

**Secretário:**
```json
{
	"token": "seu_token_api",
	"name": "Carlos Oliveira",
	"email": "secretario@teste.com",
	"registration_number": "11223344",
	"access_token": "xyz789uvw012",
	"profile_id": 3
}
```

**Aluno:**
```json
{
	"token": "seu_token_api",
	"name": "Ana Costa",
	"email": "aluno@teste.com",
	"registration_number": "99887766",
	"access_token": "qwe456rty789",
	"profile_id": 4
}
```

**Advogado (com OAB):**
```json
{
	"token": "seu_token_api",
	"name": "Pedro Souza",
	"email": "advogado@teste.com",
	"registration_number": "55443322",
	"access_token": "asd123fgh456",
	"profile_id": 5,
	"oab": "654321",
	"oab_uf": "RJ"
}
```

**Estagiário:**
```json
{
	"token": "seu_token_api",
	"name": "Julia Lima",
	"email": "estagiario@teste.com",
	"registration_number": "66778899",
	"access_token": "zxc789bnm012",
	"profile_id": 6
}
```

### Respostas

**Sucesso (200):**
```json
{
	"msg": "Aluno cadastrado com sucesso.",
	"name": "Ana Costa",
	"email": "aluno@teste.com",
	...
}
```

**Erro - Email duplicado (400):**
```json
"O email aluno@teste.com ja existe em nosso sistema. Favor trocar o email ou inativar o cadastro anterior."
```

**Erro - Matrícula duplicada (400):**
```json
"A matricula 99887766 ja existe em nosso sistema. Favor trocar a matricula ou inativar o cadastro anterior."
```

**Erro - Sem autorização (401):**
```json
"YOU_NOT_ACCESS"
```

# Atualizar um usuário

> **Método:** POST
> **URL:** https://instancia.bonsae.com.br/api/webhook/update-user
> **Autenticação:** Token obrigatório

### Parâmetros de identificação (obrigatório um dos dois 'oldEmail' ou 'oldRegistrationNumber' para identificar o usuário)
| Variável | Tipo | Descrição |
| ------ | ------ | ------ |
| `token*` | string | Token de autenticação da API |
| `oldEmail` | string | Email atual do usuário (para identificá-lo) |
| `oldRegistrationNumber` | string | Matrícula atual do usuário (para identificá-lo) |

### Parâmetros para atualização (opcionais)
| Variável | Tipo | Descrição |
| ------ | ------ | ------ |
| `name` | string | Novo nome do usuário |
| `password` | string | Nova senha do usuário |
| `email` | string | Novo email do usuário |
| `registration_number` | string | Nova matrícula do usuário |
| `access_token` | string | Novo token de autenticação |
| `profile_id` | integer | Novo identificador de perfil |
| `oab` | string | Número da OAB |
| `oab_uf` | string | UF da OAB |

### Exemplo de requisição
```json
{
	"token": "seu_token_api",
	"oldEmail": "user@teste.com",
	"name": "Nome Atualizado",
	"registration_number": "87654321",
	"password": "nova_senha",
	"profile_id": 2
}
```

### Respostas

**Sucesso (200):**
```json
{
	"name": "Nome Atualizado",
	"registration_number": "87654321",
	...
}
```

**Erro - Usuário não encontrado (400):**
```json
"Nenhum usuário encontrado com esse email: user@teste.com"
```

**Erro - Email/matrícula já existe (400):**
```json
"A matricula 87654321 ou email novo@teste.com ja existe em nosso sistema."
```

# Consultar um usuário

> **Método:** POST
> **URL:** https://instancia.bonsae.com.br/api/webhook/user
> **Autenticação:** Token obrigatório

### Parâmetros (obrigatório um dos dois)
| Variável | Tipo | Descrição |
| ------ | ------ | ------ |
| `token*` | string | Token de autenticação da API |
| `email` | string | Email do usuário |
| `registration_number` | string | Matrícula do usuário |

### Exemplo de requisição
```json
{
	"token": "seu_token_api",
	"email": "bob@teste.com"
}
```

ou

```json
{
	"token": "seu_token_api",
	"registration_number": "12345678"
}
```

### Respostas

**Sucesso (200):**
```json
{
	"id": 123,
	"name": "Bob Silva",
	"profile_id": 4,
	"active": true,
	"email": "bob@teste.com",
	"registration_number": "12345678",
	"telephone": "(11) 98765-4321",
	"cpf": "123.456.789-00",
	"oab": null,
	"oab_uf": null,
	"period_id": 1
}
```

**Erro - Usuário não encontrado (400):**
```json
"Usuario nao encontrado com os dados informados."
```

**Erro - Parâmetros faltando (400):**
```json
"Favor enviar email ou registration_number(matricula), no corpo da consulta."
```

# Listar todos os usuários

> **Método:** POST
> **URL:** https://instancia.bonsae.com.br/api/webhook/all-users
> **Autenticação:** Token obrigatório

### Parâmetros
| Variável | Tipo | Descrição |
| ------ | ------ | ------ |
| `token*` | string | Token de autenticação da API |

### Exemplo de requisição
```json
{
	"token": "seu_token_api"
}
```

### Respostas

**Sucesso (200):**
```json
[
	{
		"id": 1,
		"name": "João Silva",
		"registration_number": "12345678",
		"email": "joao@teste.com",
		"profile_id": 1
	},
	{
		"id": 2,
		"name": "Maria Santos",
		"registration_number": "87654321",
		"email": "maria@teste.com",
		"profile_id": 2
	}
]
```

# Deletar um usuário

> **Método:** DELETE
> **URL:** https://instancia.bonsae.com.br/api/webhook/delete-user
> **Autenticação:** Token obrigatório

### Parâmetros (obrigatório um dos dois 'email' ou 'registration_number')
| Variável | Tipo | Descrição |
| ------ | ------ | ------ |
| `token*` | string | Token de autenticação da API |
| `email` | string | Email do usuário |
| `registration_number` | string | Matrícula do usuário |

### Exemplo de requisição
```json
{
	"token": "seu_token_api",
	"email": "usuario@teste.com"
}
```

### Respostas

**Sucesso (200):**
```json
"Usuario deletado com sucesso."
```

**Erro - Usuário não encontrado (400):**
```json
"Usuario nao encontrado com os dados informados."
```

# Códigos de erro

A API retorna as seguintes mensagens de erro:

| Código HTTP | Mensagem | Descrição |
| ------ | ------ | ------ |
| 400 | `ERROR_IN_REQUEST` | Erro genérico na requisição |
| 401 | `YOU_NOT_ACCESS` | Token de autenticação inválido ou ausente |

[//]: # (These are reference links used in the body of this note and get stripped out when the markdown processor does its job. There is no need to format nicely because it shouldn't be seen. Thanks SO - http://stackoverflow.com/questions/4823468/store-comments-in-markdown-syntax)

   [dill]: <https://github.com/joemccann/dillinger>
   [git-repo-url]: <https://github.com/joemccann/dillinger.git>
   [john gruber]: <http://daringfireball.net>
   [df1]: <http://daringfireball.net/projects/markdown/>
   [markdown-it]: <https://github.com/markdown-it/markdown-it>
   [Ace Editor]: <http://ace.ajax.org>
   [node.js]: <http://nodejs.org>
   [Twitter Bootstrap]: <http://twitter.github.com/bootstrap/>
   [jQuery]: <http://jquery.com>
   [@tjholowaychuk]: <http://twitter.com/tjholowaychuk>
   [express]: <http://expressjs.com>
   [AngularJS]: <http://angularjs.org>
   [Gulp]: <http://gulpjs.com>

   [PlDb]: <https://github.com/joemccann/dillinger/tree/master/plugins/dropbox/README.md>
   [PlGh]: <https://github.com/joemccann/dillinger/tree/master/plugins/github/README.md>
   [PlGd]: <https://github.com/joemccann/dillinger/tree/master/plugins/googledrive/README.md>
   [PlOd]: <https://github.com/joemccann/dillinger/tree/master/plugins/onedrive/README.md>
   [PlMe]: <https://github.com/joemccann/dillinger/tree/master/plugins/medium/README.md>
   [PlGa]: <https://github.com/RahulHP/dillinger/blob/master/plugins/googleanalytics/README.md>

