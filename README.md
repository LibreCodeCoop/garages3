# garages3
Instruções para rodar o garages3 em docker.


## Configurações
- Defina as variáveis no arquivo `garage.toml`:
    - `rpc_secret` - Senha utilizada pra ingressar no cluster.
    - `rpc_public_addr` - Endereço do servidor que está sendo configurado

- `rpc_secret` deve ser uma variável de 32 bits em hexadecimal. Pode gerar uma com o comando: `openssl rand -hex 32`


## Controlando o daemon
- Crie um `alias` e adicione ao `~/.bashrc_alias`:

    `alias garage="docker exec -ti <nome do container > /garage"`


