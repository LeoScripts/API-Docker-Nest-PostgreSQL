Deploy!

Enfim chegamos à última parte dessa série: Vamos colocar nossa API na nuvem! A primeira coisa que precisamos para colocar nossa API online é de um servidor onde hospedá-la. Nesse tutorial utilizarei os serviços da DigitalOcean. Caso ainda não tenha, você pode criar uma conta aqui. Criando uma conta através desse link você ganha $100 em créditos para serem gastos em 60 dias na plataforma.
<br>

Após criar sua conta e acessar a plataforma, vamos criar uma droplet. Droplets são a maneira como a DigitalOcean chama os servidores em nuvem (famosos VPS — Virtual Private Server).
<br>
Para criar um droplet clique no dropdown Create, no canto direito superior, e selecione “Droplets”, a primeira opção.
<br>
Vamos agora selecionar o Sistema Operacional do nosso servidor:
<br>
Nesse tutorial vamos seguir com o Ubuntu mesmo. Entretanto, temos algumas outras opções de SO, dependendo das necessidades do seu sistema.
<br>
Reparem também na existência da aba “Marketplace”. Nela você encontra algumas imagens de criação de droplets que já vêm com uma série de funcionalidades, como o stack LAMP instalado, docker, Wordpress, entre várias outras coisas. Vale a pena conferir!
<br>
Escolhido a imagem de criação, vamos para a segunda etapa: A escolha do plano.
<br>
Para este tutorial vamos escolher o plano mais básico de todos, de $5/mês. Vale a pena dar uma olhada nos outros planos disponíveis para uso futuro.
<br>
A próxima etapa é opcional: Adicionar um Volume à nossa Droplet. Pense em um Volume como espaço adicional em disco para sua Droplet. No nosso caso os 25GB são mais do que suficiente.
<br>
Para este tutorial vamos escolher o plano mais básico de todos, de $5/mês. Vale a pena dar uma olhada nos outros planos disponíveis para uso futuro.A próxima etapa é opcional: Adicionar um Volume à nossa Droplet. Pense em um Volume como espaço adicional em disco para sua Droplet. No nosso caso os 25GB são mais do que suficiente.
<br>
A próxima etapa também é opcional: Você pode escolher habilitar alguns recursos adicionais para sua Droplet, nesse caso não vamos habilitar nenhum deles.
<br>
Agora vamos escolher a forma pela qual ser autenticados para acessar nossa Droplet: podemos escolher autenticação via chave SSH ou usuário/senha. Nesse tutorial vou utilizar a chave SSH, mas vou explicar como ambos funcionam.
<br>
Caso você opte por usuário/senha você deve marcar a segunda opção:
<br>
Escolhendo essa opção, ao finalizar a criação da Droplet você receberá no seu email uma senha para acessar a Droplet recém criada com o usuário root.
<br>
Caso opte por autenticação via chave SSH, deve escolher a primeira opção:
<br>
Repare no botão New SSH Key. É através dele que você adicionará uma chave SSH para ser utilizada na autenticação. Você pode aprender como criar uma chave SSH e utilizá-la como método de autenticação [aqui](https://help.github.com/pt/github/authenticating-to-github/generating-a-new-ssh-key-and-adding-it-to-the-ssh-agent).
<br>
Com sua chave SSH criada, basta ir na pasta onde você a salvou (normalmente ~/.ssh) e copiar o conteúdo da chave pública (possui extensão .pub, normalmente se chama id_rsa.pub, se você escolheu a opção padrão durante a criação) e colar esse mesmo conteúdo na janela que irá abrir ao clicar em New SSH Key. Você também deve dar um nome para identificar essa chave no painel da DigitalOcean.
<br>
Agora podemos finalizar a criação da nossa Droplet:
<br>
Você tem a opção de criar mais de uma Droplet com as mesmas configurações, bem como a opção de adicionar um nome mais amigável a ela (eu escolhi tutorial-medium, normalmente esse campo vem preenchido com um nome padrão gerado pela DigitalOcean). Você também pode adicionar tags para ajudar a localizar sua Droplet no futuro, caso venha a ter várias Droplets na sua conta.
<br>
Vale destacar, por último, a opção de backups. Um backup sempre adiciona 20% ao valor mensal de sua Droplet e com ele você tem a vantagem de ter um backup semanal automático do seu sistema. Muito útil para ambientes de produção, mas desnecessário nessa fase de desenvolvimento.
<br>
Vamos clicar em Create Droplet para finalizar. Isso pode levar alguns minutos.
<br>
Finalizado o processo, você estará numa tela:
<br>
Agora você pode acessar sua droplet através do endereço de IP informado. Vamos abrir o terminal e executar o seguinte comando:
```
ssh root@ip-da-droplet
```
ex:
```
ssh root@67.205.181.243
```
> Se você escolheu autenticação por chave SSH você acessará sua Droplet imediatamente. Caso tenha escolhido autenticação via usuário/senha será pedido que você informe a senha que você recebeu via email, bem como será pedido que crie uma nova senha logo em seguida.

>>Recomendo que sigam [esse tutorial](https://www.digitalocean.com/community/tutorials/initial-server-setup-with-ubuntu-18-04) da DigitalOcean para configuração inicial de um servidor.
<br>
No nosso caso, vamos seguir com o usuário root padrão mesmo, mas fica o alerta que <b> isso não é recomendado em ambiente de produção</b>. Estamos utilizando ele apenas para testes, em um ambiente de desenvolvimento.
<br>
Como estamos utilizando o docker, vamos instalá-lo na nossa droplet:
```
curl -fsSL https://get.docker.com -o get-docker.sh
sh get-docker.sh
rm get-docker.sh
```
<br>
Em seguida, vamos instalar o docker-compose. Acesse o link do github para ver qual a release mais recente, bem como encontrar os comandos para instalação da mesma. No nosso caso, a release mais recente no instante em que escrevia esse tutorial era a 1.25.4. Então a instalação fica assim:

```
curl -L https://github.com/docker/compose/releases/download/1.25.4/docker-compose-`uname -s`-`uname -m` -o /usr/local/bin/docker-compose
```

<br>
Em seguida:
```
chmod +x /usr/local/bin/docker-compose
```
E então, para verificar se está funcionando:
```
docker-compose --version
```
A saída deve ser algo como:
```
docker-compose version 1.25.4, build 8d51620a
```
>Detalhe: Já aconteceu comigo algumas vezes do comando simplesmente travar e não exibir nenhum resultado. Algumas vezes a memória de 1GB da Droplet padrão não é o suficiente para executar o comando. Para o ambiente de testes, vamos adicionar um arquivo de swap à nossa Droplet:
<br>
```
sudo fallocate -l 2G /swapfile
sudo chmod 600 /swapfile
sudo mkswap /swapfile
sudo swapon /swapfile
```
Por último, precisamos editar o arquivo `/etc/fstab` para que a alteração seja permanente:
```
sudo nano /etc/fstab
```
E adicionar ao final do arquivo:
```
/swapfile swap swap defaults 0 0
```
Pressione ctrl+o para salvar e ctrl+x para sair.

Execute o comando novamente:

` docker-compose --version`

E agora deverá receber a saída desejada.


>Agora vamos às alterações que devem ser feitas no nosso projeto para prosseguirmos com o deploy no servidor.

- Primeiro, vamos adicionar um Dockerfile para inicializar o container da nossa API, na raiz do nosso projeto (fora de src):
```
FROM node:alpine

# diretório alvo
RUN mkdir -p /usr/src/node-api
WORKDIR /usr/src/node-api

# instalação de dependências
RUN apk update && apk upgrade
RUN apk add python3 g++ make

# copiar o projeto e instalar os pacotes com o npm
COPY . /usr/src/node-api/
RUN npm install

# instalação dos pacotes para envio de email
RUN apk add msmtp
RUN ln -sf /usr/bin/msmtp /usr/sbin/sendmail

# abrindo a porta 3000
EXPOSE 3000

# inicializando a API
CMD [ "npm","run", "start:dev" ]
```

Repassando o que o Dockerfile está especificando:

- Vamos partir da imagem node:alpine;
- Criamos um diretório em /usr/src chamado node-api;
- Definimos o diretório recém criado como nosso diretório atual;
- Instalamos possíveis atualizações do nosso sistema operacional, no caso o alpine;
- Instalamos o pyhton3, g++ e o make, que são necessários para o bom funcionamento do npm install;
- Copiamos o conteúdo do nosso diretório atual para o diretório recém criado dentro do container;
- Rodamos o npm install;
- Instalamos mais alguns pacotes, dessa vez o msmtp, equivalente ao sendmail para o alpine, necessário para enviarmos emails;
- Criamos um atalho para o msmtp com o nome de sendmail para que nossa aplicação o execute quando tentar executar o sendmail;
- Abrimos a porta 3000
- Rodamos o comando npm run start:dev para inicializar a API dentro do container.
>Vamos criar também um arquivo `.dockerignore` no mesmo local:
```
node_modules
logs
Dockerfile
docker-compose.yml
```

>Esse arquivo especifica quais pastas/arquivos devem ser ignorados pelo comando COPY do Dockerfile.

- Agora vamos adicionar o serviço da nossa API, especificado no Dockerfile, ao `docker-compose.yml`:
```
version: '3'

services:
  api:
    build:
      context: .
      dockerfile: api.Dockerfile
    ports:
      - '3000:3000'
    container_name: api
    restart: always
    volumes:
      - api-logs:/usr/src/node-api/logs:rw
    environment:
      - NODE_ENV=development
    depends_on:
      - pgsql

  pgsql:
    image: postgres:alpine
    ports:
      - '5432:5432'
    container_name: 'pgsql'
    restart: always
    volumes:
      - pg-data:/var/lib/postgresql/data
    environment:
      POSTGRES_USER: pguser
      POSTGRES_PASSWORD: pgpassword
      POSTGRES_DB: nestjs

  adminer:
    image: adminer
    restart: always
    ports:
      - 8080:8080

volumes:
  api-logs:
  pg-data:

```

>Reparem que dessa vez, ao invés de especificarmos uma imagem do docker hub, especificamos para o docker-compose construir uma imagem utilizando o api.Dockerfile e utilizá-la para montar o container.

- Por último, vamos alterar o arquivo `src/configs/typeorm.config.ts`:
```
import { TypeOrmModuleOptions } from '@nestjs/typeorm';

export const typeOrmConfig: TypeOrmModuleOptions = {
  type: 'postgres',
  host: 'pgsql',
  port: 5432,
  username: 'pguser',
  password: 'pgpassword',
  database: 'nestjs',
  entities: [__dirname + '/../**/*.entity.{js,ts}'],
  synchronize: true,
};
```
>Essa alteração foi bem pequena: alteramos o host do banco de dados de local host para pgsql, que é a forma como o container da API vai conseguir acessar o container do banco de dados. Vamos subir essas alterações para o Git (se você clonar o repositório da parte 7 do meu GitHub ele já virá com essas alterações, apenas especifiquei elas aqui para que entendam o que está acontecendo).

- Vamos voltar ao nosso servidor e clonar o repositório da nossa aplicação:
```
git clone https://github.com/iagomaia/nestjs-medium-parte7
cd nestjs-medium-parte7
```

Agora vamos subir nossos containers:
```
docker-compose up
```

Essa será a saída ao finalizar:

Alguns detalhes:

- Você poderá realizar requisições para o endereço IP da sua droplet, na porta 3000.
- Para prevenir o uso das Droplets como meio de envio de emails de spam a DigitalOcean bloqueia automaticamente as portas necessárias para o envio de emails por 60 dias após a criação da Droplet, então nossos serviços de envio de emails não irão funcionar.
- Você pode enviar emails normalmente se utilizar serviços como o Sendgrid ou o Mailgun.
- Você também pode solicitar via ticket o desbloqueio de sua Droplet, mas isso leva um certo tempo.
- Ao acessar o endereço de sua Droplet no navegador na porta 8080 você terá acesso ao Adminer para visualizar seu banco de dados.

## E com isso finalizamos 