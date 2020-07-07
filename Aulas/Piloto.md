# Aula teórica - Piloto

Tema: Manipulação dos métodos da API-TELEGRAM-BOT


Para começar, precisamos entender primeiro como a API-TELEGRAM-BOT funciona e, como devemos "conversar com ela"; vamos nessa aula entender como iremos fazer isso onde estarei fragmentando algumas coisa para ficar facil de entendermos.

### Primeiramente, o que é uma API?

*"API é uma Interface de Programação de Aplicação (pt-BR), cuja sigla API provém do Inglês Application Programming Interface, é um conjunto de rotinas e padrões estabelecidos por um software para a utilização das suas funcionalidades por aplicativos que não pretendem envolver-se em detalhes da implementação do software, mas apenas usar seus serviços.*

*De modo geral, a API é composta por uma série de funções acessíveis somente por programação, e que permitem utilizar características do software menos evidentes ao utilizador tradicional... ([continuar lendo](https://pt.wikipedia.org/wiki/Interface_de_programa%C3%A7%C3%A3o_de_aplica%C3%A7%C3%B5es))"*

Para entendermos melhor, estarei colocando dentro do contesto que queremos...

Por exemplo, quando falamos sobre o uso da API-TELEGRAM-BOT, estamos dizendo que usaremos a funcionalidade do aplicativo bot para executar funções que poderíamos realizar sendo apenas um usuário comum, mas estamos executando como um bot, por que esse é o padrão para qualquer automação.

### Como utilizar API / Como "Conversar" com a API?

No site oficial é apresentado que todas as consultas/"conversas" para a API-TELEGRAM-BOT devem ser enviadas via HTTPS e necessariamente devem ser apresentadas desta maneira: `https://api.telegram.org/bot<token>/METHOD_NAME`. 

Essas consultas/"conversas" com a API, são chamadas de requisições, essas requisições pode ser enviadas via GET ou POST, desde que respeite todos os parâmetros obrigatórios de cada método.

Vou citar um exemplo de como devemos solicitar os dados da pela api, abaixo estou deixando um link clique para acessar, ao carregar a pagina troque `<token>` pela sua token que te instrui a criar na introdução.

##### Exemplo:
`link:` [https://api.telegram.org/bot<token>/getMe](https://api.telegram.org/bot<token>/getMe)

`resultado:`
```json
{
"ok":true,
"result": {
  "id":523403928,
  "is_bot":true,
  "first_name":
  "Nome do seu bot",
  "username":"Nomedoseubot",
  "can_join_groups":true,
  "can_read_all_group_messages":false,
  "supports_inline_queries":false
  }
}
```

Pronto, acabamos de enviar uma requisição HTTP via GET a API-TELEGRAM-BOT, porem não iremos acessar sempre nosso navegador para enviar uma requisição, certo? e agora que a programação entra

Precisamos criar uma função para fazer essa conversação para gente sem a necessidade de abrirmos um pagina toda vez que precisarmos enviar uma requisição.

Vamos lá, precisamos primeiro definir uma lib para trabalharmos, vou usar a lib `requests` do python, por experiencia eu vou trabalhar com class porque acho melhor para desenvolver o que preciso, mas você pode criar funções, como disse na introdução o objetivo desse projeto não é programação, prosseguindo:

Primeiro instalarmos a lib em nosso terminal, então abrar o terminal e execute:`sudo pip3 install requests`, logo em seguida abra um editor de texto e digite: 
```python3
import requests
def sendRequest(url, params=None, headers=None, files=False, post=False):
	try:
		requests.get(url, params=params, headers=headers, files=files, post=post)
	except Exception as error:
		print(error)
```
Nessa função acima estamos tratando apenas a utilização da lib `requests`, com essa função já conseguimos executar requisições em qualquer HTTP, mas o objetivo é trabalhar com a API-TELEGRAM-BOT, ou seja, precisa inserir dados para comunicarmos com ela, o que precisamos para isso?

Sabemos que a API obrigatoriamente precisa que informarmos sempre a token do bot e que ela suporta quatro maneiras de passar parâmetros para os métodos, sendo: 
```
    String de consulta da URL (que fizemos logo acima pelo navegador)
    application/x-www-form-urlencoded
    application/json (não é recomendado usar para fazer upload de arquivos)
    multipart/form-data (recomendado usar para fazer upload de arquivos)
```

Com base nisso, precisamos definir a token do bot, os cabeçalhos (headers) para passarmos parâmetros para os métodos e por fim uma maneira de usar a função `sendRequest`, estou deixando abaixo a solução que fiz para isso com comentario.

```python3
import os, requests, json
class Method():
	"""
	Este método ajudará você a manipular a API do Telegram facilmente, 
	porque basicamente ela se comunicará com a API-TELEGRAM-BOT enviando os argumentos necessários
	sem a necessidade de escrever muito código ou usar uma Frameworks/SDK/Wrapper para o Telegram.
	"""
	def __init__(self):
		self.token = os.environ['SECRET_KEY'] # Definindo token do bot
		self.apitelegram = f"https://api.telegram.org/bot{self.token}" # Definindo api-telegram-bot
		self.headers = {'content-type': 'application/json', 'Cache-Control': 'no-cache'} # Definindo headers
	@staticmethod
	def sendRequest(url, params=None, headers=None, files=False, post=False):
		"""
		Esta função, quando chamada, executará uma requisição na URL fornecida
		com os argumentos definidos.
		Nota: Não server somente para trabalhar com a api do Telegram
		"""
		try:
			if (post): #caso precise enviar arquivos, essa condição será usada.
				data = requests.post(url, params=params, headers=headers, files=files, post=post)
			else:
				#caso não precise enviar arquivos, essa condição será usada.
				data = requests.get(url, params=params, headers=headers)
		except Exception as error:
			print(error); data = False

		if (data != False):
			if (data.status_code == 200): 
				return dict(success=True, code=data.status_code, response=data.json())
			else:
				return dict(success=False, code=data.status_code, response=data.json())
		else:
			return dict(success=False, code=404, response=False)

	def sendTG(self, method="sendMessage", strfile=False, file=False, **args):
		"""
		Esta função, quando chamada, executará uma requisição 
		que enviará os argumentos para o API-Telegram-BOT.
		Legenda:
			method: Nome do method
			strfile: quando for necessario enviar um arquivo, nome do arquivo será indexado aqui
			file: quando for necessario enviar um arquivo, o arquivo será indexado aqui
			args: para os demais argumentos, depende muito do method
			então ele irá coletar o que chegar por aqui
		"""
		if (strfile != False) and (file != False):
			#Caso as condições strfile e file, não forem falsas, essa condição será usada

			return self.sendRequest(f"{self.apitelegram}/{method}", params=locals()['args'], headers=self.headers, files=dict(strfile=file), post=True)
		elif (strfile == False) and (file == False):
			#Caso as condições strfile e file, forem falsas, essa condição será usada
			
			return self.sendRequest(f"{self.apitelegram}/{method}", params=locals()['args'], headers=self.headers)
```
Não vou entrar em muitos detalhes, pois deixei o codigo muito bem comentado, mas quero que você entenda como usar o código acima, então estou deixando alguns exemplos abaixo sobre como usá-lo

Primeiro oriento a salvá-lo como `method.py`, logo em seguida apenas para teste, abra seu terminal, navegue até a pasta em que o arquivo foi salvo e execute: `python3`, agora agora siga o exemplo abaixo (Nota: você precisa definir sua token no local informado).

```
Exemplo de como usar:
from name_file import Method
api = Method()
api.sendTG(method="getME")
api.sendTG(chat_id=438131290,text='oi')
api.sendTG(method="sendPhoto", chat_id=438131290, photo="https://img.olhardigital.com.br/uploads/acervo_imagens/2020/04/r4x3/20200423030657_660_495_-_python.jpg", caption='<b>ping</b>', parse_mode='HTML')

Outro exemplo:
from name_file import Method
import os, gtts
api = Method()
try:
	AUDIO = gtts.gTTS(cmd[1], lang="en")
	AUDIO.save('audio.ogg')
	AUDIO = open('audio.ogg', 'rb')
	api.sendTG(method="sendAudio", strfile="audio", file=AUDIO, chat_id=438131290)
except Exception as error:
	print(error)
finally:
	AUDIO.close()
	os.remove('audio.ogg')
```


Pronto!!!! chegamos no fim desta aula, entendemos como uma API funciona e como conversar com ela, aplicando isso no nosso objetivo.

Estarei deixando o arquivo criado nessa aula na pasta: **Projeto**, pois usaremos muito esse arquivo daqui para frente.