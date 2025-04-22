# garages3
Instruções para rodar o garages3 em docker.


## Configurações
- Defina as variáveis no arquivo `garage.toml`:
    - `rpc_secret` - Senha utilizada pra ingressar no cluster.
    - `rpc_public_addr` - Endereço do servidor que está sendo configurado

- `rpc_secret` deve ser uma variável de 32 bits em hexadecimal. Pode gerar uma com o comando: `openssl rand -hex 32`


- Inicie o serviço:

    `docker compose up -d`

## Controlando o daemon
- Crie um `alias` e adicione ao `~/.bashrc_alias`:

    `alias garage="docker exec -ti <nome do container > /garage"`

- Executando o comando `garage status` é possível ver status de aplicação.
    ```bash
    2025-04-22T12:36:32.002383Z  INFO garage_net::netapp: Connected to 189.48.251.22:3901, negotiating handshake...    
    2025-04-22T12:36:32.043613Z  INFO garage_net::netapp: Connection established to 3f8732ea76d76654    
    ==== HEALTHY NODES ====
    ID                Hostname                      Address              Tags  Zone  Capacity          DataAvail
    3f8732ea7676654  servidor.dominio.com.br  189.48.251.22:3901              NO ROLE ASSIGNED
    ```

## Conectando a outros nós
- A primeira vez que o Garage iniciar, será gerado um identificador único baseado na chave pública e privada.
- Para obter o identificar, execute o comando `garage node id`.
    ```bash
    Terra# garage node id
    3f8732ea76d769549fa26a7d7581c5544e33fe438d9ee591bf5d646618a5097026@189.48.251.22:3901
    ```

- Para instruir outro servidor a se conectar, execute:
    ```bash
    Lua# garage node connect 3f8732ea76d769549fa26a7d7581c5544e33fe438d9ee591bf5d646618a5097026@189.48.251.22:3901
    ```

- Não é necessário instruir todos nós a se conectarem a todos os outros nós: os nós executam um processo de auto-descoberta.


## Criando o layout de um cluster
- Criaremos um layout e informaremos ao Garage, com informações do espaço em disco de cada servidor, sua localização (zona). 
- Como exemplo, leve em consideração o layout abaixo:
    ```bash
    Localização	Nome	Espaço	Identificador	Zona (-z)	Capacidade (-c)
    Paris	    Mercury	1 TB	563e	        par1    	1T
    Paris	    Venus	2 TB	86f0	        par1	    2T
    London	    Earth	2 TB	6814	        lon1	    2T
    Brussels	Mars	1.5 TB	212f	        bru1	    1.5T
    ```

- O identificador do nó é gerado na primeira inicialização do serviço e pode ser obtido invocando o comando `garage status`. Podemos utilizar os 4 primeiros caracteres para identificar o nó.
- O comando `garage layout assign` será utilizado para configurar corretamente os parâmetros para cada nó.

### Zonas
- Zonas são simplesmente um identificador escolhido pelo usuário que identifica um grupo de servidores agrupados logicamente. Cabe ao administrador do sistema que implementa o Garage identificar o que significa "agrupados".

- Na maioria dos casos, uma zona corresponderá a uma localização geográfica (por exemplo, um datacenter). Nos bastidores, o Garage usará a definição de zona para tentar armazenar os mesmos dados em zonas diferentes, a fim de fornecer alta disponibilidade mesmo com falhas em uma zona.

- As zonas são passadas para o Garage usando o sinalizador `-z` do comando `garage layout assign` (veja abaixo).

### Capacidade
- O Garage precisa saber a capacidade (espaço em disco) que é possível utilizar em cada nó, para balancear os dados corretamente.
- A capacidade é expressada em `bytes` e são passadas utilizando o parâmetro `-c`.

### Tags
- Podem ser adicionadas para indicar informações extras e não tem nenhum significado para o Garage.

### Configurando a topologia
- Dada a tabela de exemplo utilizada, podemos configurar o cluster como a seguir:
    ```bash
    garage layout assign 563e -z par1 -c 1T -t mercury
    garage layout assign 86f0 -z par1 -c 2T -t venus
    garage layout assign 6814 -z lon1 -c 2T -t earth
    garage layout assign 212f -z bru1 -c 1.5T -t mars
    ```
- Para verificar as mudanças: `garage layout show`
- Aplicando as mudanças:
    ```bash
    garage layout apply
    ```

## Utilização
- Com o Garage é possível criar buckets, gerenciar chaves de acesso, hospedar sites, entre outros. Tudo isso é feito utilizando a linha de comando invocando `garage`.
- A utilização da linha de comando é bem documentada, com o parâmetro `--help` e o subcomando `help` (por exemplo, `garage help`, `garage key --help`).


# Fontes
-  [Documentação oficial](https://garagehq.deuxfleurs.fr/documentation/cookbook/real-world/)