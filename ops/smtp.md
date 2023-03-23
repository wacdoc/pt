Entre no warehouse de configuração ops.soft, execute `./ssl.sh` e uma pasta `conf` será criada no **diretório superior** .

## preâmbulo

O SMTP pode adquirir serviços diretamente de fornecedores de nuvem, como:

* [Amazon SES SMTP](https://docs.aws.amazon.com/ses/latest/dg/send-email-smtp.html)
* [Empurrão de e-mail na nuvem Ali](https://www.alibabacloud.com/help/directmail/latest/three-mail-sending-methods)

Você também pode construir seu próprio servidor de e-mail - envio ilimitado, baixo custo geral.

Abaixo, demonstramos passo a passo como construir nosso próprio servidor de correio.

## seleção de servidor

O servidor SMTP auto-hospedado requer um IP público com as portas 25, 456 e 587 abertas.

As nuvens públicas comumente usadas bloqueiam essas portas por padrão e pode ser possível abri-las emitindo uma ordem de serviço, mas é muito problemático, afinal.

Eu recomendo comprar de um host que tenha essas portas abertas e suporte a configuração de nomes de domínio reverso.

Aqui, eu recomendo [o Contabo](https://contabo.com) .

Contabo é um provedor de hospedagem com sede em Munique, Alemanha, fundado em 2003 com preços muito competitivos.

Se você escolher o Euro como moeda de compra, o preço será mais barato (um servidor com 8 GB de memória e 4 CPUs custa cerca de 529 yuans por ano e a taxa de instalação inicial é gratuita por um ano).

![](https://pub-b8db533c86124200a9d799bf3ba88099.r2.dev/2023/03/UoAQkwY.webp)

Ao fazer um pedido, observe `prefer AMD` , e o servidor com CPU AMD terá melhor desempenho.

A seguir, usarei o VPS da Contabo como exemplo para demonstrar como construir seu próprio servidor de correio.

## Configuração do sistema Ubuntu

O sistema operacional aqui é o Ubuntu 22.04

![](https://pub-b8db533c86124200a9d799bf3ba88099.r2.dev/2023/03/smpIu1F.webp)

![](https://pub-b8db533c86124200a9d799bf3ba88099.r2.dev/2023/03/m7Mwjwr.webp)

Se o servidor em ssh exibir `Welcome to TinyCore 13!` (conforme figura abaixo), significa que o sistema ainda não foi instalado. Desconecte o ssh e aguarde alguns minutos para fazer login novamente.

![](https://pub-b8db533c86124200a9d799bf3ba88099.r2.dev/2023/03/-qKACz9.webp)

Quando `Welcome to Ubuntu 22.04.1 LTS` for exibido, a inicialização estará concluída e você poderá continuar com as etapas a seguir.

### [Opcional] Inicialize o ambiente de desenvolvimento

Esta etapa é opcional.

Por conveniência, coloquei a instalação e configuração do sistema do software ubuntu em [github.com/wactax/ops.os/tree/main/ubuntu](https://github.com/wactax/ops.os/tree/main/ubuntu) .

Execute o seguinte comando para instalar com um clique.

```
bash <(curl -s https://raw.githubusercontent.com/wactax/ops.os/main/ubuntu/boot.sh)
```

Usuários chineses, por favor, usem o seguinte comando, e o idioma, fuso horário, etc. serão definidos automaticamente.

```
CN=1 bash <(curl -s https://ghproxy.com/https://raw.githubusercontent.com/wactax/ops.os/main/ubuntu/boot.sh)
```

### Contabo permite IPV6

Ative o IPV6 para que o SMTP também possa enviar e-mails com endereços IPV6.

edite `/etc/sysctl.conf`

Modifique ou adicione as seguintes linhas

```
net.ipv6.conf.all.disable_ipv6 = 0
net.ipv6.conf.default.disable_ipv6 = 0
net.ipv6.conf.lo.disable_ipv6 = 0
```

Continue com [o tutorial do contabo: Adicionando conectividade IPv6 ao seu servidor](https://contabo.com/blog/adding-ipv6-connectivity-to-your-server/)

Edite `/etc/netplan/01-netcfg.yaml` , adicione algumas linhas conforme mostrado na figura abaixo (o arquivo de configuração padrão do Contabo VPS já possui essas linhas, basta descomentá-las).

![](https://pub-b8db533c86124200a9d799bf3ba88099.r2.dev/2023/03/5MEi41I.webp)

Em seguida, `netplan apply` para fazer com que a configuração modificada entre em vigor.

Depois que a configuração for bem-sucedida, você pode usar `curl 6.ipw.cn` para visualizar o endereço ipv6 de sua rede externa.

## Clone as operações do repositório de configuração

```
git clone https://github.com/wactax/ops.soft.git
```

## Gere um certificado SSL gratuito para o seu nome de domínio

Enviar e-mail requer um certificado SSL para criptografia e assinatura.

Usamos [acme.sh](https://github.com/acmesh-official/acme.sh) para gerar certificados.

acme.sh é uma ferramenta de assinatura de certificado automatizada de código aberto,

Entre no warehouse de configuração ops.soft, execute `./ssl.sh` e uma pasta `conf` será criada no **diretório superior** .

Encontre seu provedor de DNS em [acme.sh dnsapi](https://github.com/acmesh-official/acme.sh/wiki/dnsapi) , edite `conf/conf.sh` .

![](https://pub-b8db533c86124200a9d799bf3ba88099.r2.dev/2023/03/Qjq1C1i.webp)

Em seguida, execute `./ssl.sh 123.com` para gerar certificados `123.com` e `*.123.com` para seu nome de domínio.

A primeira execução instalará automaticamente [o acme.sh](https://github.com/acmesh-official/acme.sh) e adicionará uma tarefa agendada para renovação automática. Você pode ver `crontab -l` , existe uma linha como a seguir.

```
52 0 * * * "/mnt/www/.acme.sh"/acme.sh --cron --home "/mnt/www/.acme.sh" > /dev/null
```

O caminho para o certificado gerado é algo como `/mnt/www/.acme.sh/123.com_ecc。`

A renovação do certificado chamará o script `conf/reload/123.com.sh` , edite este script, você pode adicionar comandos como `nginx -s reload` para atualizar o cache do certificado de aplicativos relacionados.

## Construir servidor SMTP com chasquid

[chasquid](https://github.com/albertito/chasquid) é um servidor SMTP de código aberto escrito em linguagem Go.

Como um substituto para os antigos programas de servidor de correio como Postfix e Sendmail, o chasquid é mais simples e fácil de usar, e também é mais fácil para desenvolvimento secundário.

Execute `./chasquid/init.sh 123.com` será instalado automaticamente com um clique (substitua 123.com pelo seu nome de domínio de envio).

## Configurar assinatura de e-mail DKIM

O DKIM é usado para enviar assinaturas de e-mail para evitar que as cartas sejam tratadas como spam.

Depois que o comando for executado com sucesso, você será solicitado a definir o registro DKIM (conforme mostrado abaixo).

![](https://pub-b8db533c86124200a9d799bf3ba88099.r2.dev/2023/03/LJWGsmI.webp)

Basta adicionar um registro TXT ao seu DNS (conforme mostrado abaixo).

![](https://pub-b8db533c86124200a9d799bf3ba88099.r2.dev/2023/03/0szKWqV.webp)

## Exibir status e logs do serviço

 `systemctl status chasquid` Visualize o status do serviço.

O estado de operação normal é mostrado na figura abaixo

![](https://pub-b8db533c86124200a9d799bf3ba88099.r2.dev/2023/03/CAwyY4E.webp)

 `grep chasquid /var/log/syslog` ou `journalctl -xeu chasquid` pode visualizar o log de erros.

## Configuração de nome de domínio reverso

O nome de domínio reverso é para permitir que o endereço IP seja resolvido para o nome de domínio correspondente.

Definir um nome de domínio reverso pode impedir que os e-mails sejam identificados como spam.

Quando o e-mail for recebido, o servidor de recebimento executará a análise de nome de domínio reverso no endereço IP do servidor de envio para confirmar se o servidor de envio possui um nome de domínio reverso válido.

Se o servidor de envio não tiver um nome de domínio reverso ou se o nome de domínio reverso não corresponder ao endereço IP do servidor de envio, o servidor de recebimento poderá reconhecer o e-mail como spam ou rejeitá-lo.

Acesse [https://my.contabo.com/rdns](https://my.contabo.com/rdns) e configure conforme abaixo

![](https://pub-b8db533c86124200a9d799bf3ba88099.r2.dev/2023/03/IIPdBk_.webp)

Após definir o nome de domínio reverso, lembre-se de configurar a resolução de encaminhamento do nome de domínio ipv4 e ipv6 para o servidor.

## Edite o nome do host de chasquid.conf

Modifique `conf/chasquid/chasquid.conf` para o valor do nome de domínio reverso.

![](https://pub-b8db533c86124200a9d799bf3ba88099.r2.dev/2023/03/6Fw4SQi.webp)

Em seguida, execute `systemctl restart chasquid` para reiniciar o serviço.

## Backup conf para o repositório git

![](https://pub-b8db533c86124200a9d799bf3ba88099.r2.dev/2023/03/Fier9uv.webp)

Por exemplo, faço backup da pasta conf em meu próprio processo do github da seguinte maneira

Crie um armazém privado primeiro

![](https://pub-b8db533c86124200a9d799bf3ba88099.r2.dev/2023/03/ZSCT1t1.webp)

Entre no diretório conf e envie para o warehouse

```
git init
git add .
git commit -m "init"
git branch -M main
git remote add origin git@github.com:wac-tax-key/conf.git
git push -u origin main
```

## Adicionar remetente

correr

```
chasquid-util user-add i@wac.tax
```

Pode adicionar remetente

![](https://pub-b8db533c86124200a9d799bf3ba88099.r2.dev/2023/03/khHjLof.webp)

### Verifique se a senha está definida corretamente

```
chasquid-util authenticate i@wac.tax --password=xxxxxxx
```

![](https://pub-b8db533c86124200a9d799bf3ba88099.r2.dev/2023/03/e92JHXq.webp)

Depois de adicionar o usuário, `chasquid/domains/wac.tax/users` será atualizado, lembre-se de enviá-lo ao warehouse.

## DNS adicionar registro SPF

SPF (Sender Policy Framework) é uma tecnologia de verificação de e-mail usada para evitar fraudes por e-mail.

Ele verifica a identidade de um remetente de e-mail verificando se o endereço IP do remetente corresponde aos registros DNS do nome de domínio que afirma ser, evitando que fraudadores enviem e-mails falsos.

Adicionar registros SPF pode impedir que os e-mails sejam identificados como spam, tanto quanto possível.

Se o seu servidor de nome de domínio não suportar o tipo SPF, basta adicionar o registro do tipo TXT.

Por exemplo, o SPF de `wac.tax` é o seguinte

`v=spf1 a mx include:_spf.wac.tax include:_spf.google.com ~all`

SPF para `_spf.wac.tax`

`v=spf1 a:smtp.wac.tax ~all`

Observe que `include:_spf.google.com` aqui, porque configurarei `i@wac.tax` como o endereço de envio na caixa de correio do Google posteriormente.

## Configuração de DNS DMARC

DMARC é a abreviação de (Domain-based Message Authentication, Reporting & Conformance).

Ele é usado para capturar devoluções de SPF (talvez causadas por erros de configuração ou outra pessoa fingindo ser você para enviar spam).

Adicionar registro TXT `_dmarc` ,

O conteúdo é o seguinte

```
v=DMARC1; p=quarantine; fo=1; ruf=mailto:ruf@wac.tax; rua=mailto:rua@wac.tax
```

![](https://pub-b8db533c86124200a9d799bf3ba88099.r2.dev/2023/03/k44P7O3.webp)

O significado de cada parâmetro é o seguinte

### p (Política)

Indica como lidar com e-mails que falham na verificação SPF (Sender Policy Framework) ou DKIM (DomainKeys Identified Mail). O parâmetro p pode ser definido para um dos três valores:

* nenhum: nenhuma ação é tomada, apenas o resultado da verificação é enviado de volta ao remetente por meio do mecanismo de relatório de e-mail.
* Quarentena: Coloque o e-mail que não passou na verificação na pasta de spam, mas não rejeitará o e-mail diretamente.
* rejeitar: rejeitar diretamente e-mails que falham na verificação.

### fo (Opções de Falha)

Especifica a quantidade de informações retornadas pelo mecanismo de relatório. Pode ser definido para um dos seguintes valores:

* 0: Resultados da validação do relatório para todas as mensagens
* 1: relatar apenas mensagens que falham na verificação
* d: relatar apenas falhas de verificação de nome de domínio
* s: relatar apenas falhas de verificação de SPF
* l: relatar apenas falhas de verificação DKIM

### rua & ruf

* rua (Reporting URI for Aggregate reports): Endereço de e-mail para receber relatórios agregados
* ruf (Reporting URI for Forensic reports): endereço de e-mail para receber relatórios detalhados

## Adicionar registros MX para encaminhar e-mails para o Google Mail

Como não consegui encontrar uma caixa de correio corporativa gratuita compatível com endereços universais (Catch-All, pode receber todos os e-mails enviados para este nome de domínio, sem restrições de prefixos), usei o chasquid para encaminhar todos os e-mails para minha caixa de correio do Gmail.

**Se você tiver sua própria caixa de correio comercial paga, não modifique o MX e pule esta etapa.**

Edite `conf/chasquid/domains/wac.tax/aliases` , defina a caixa de correio de encaminhamento

![](https://pub-b8db533c86124200a9d799bf3ba88099.r2.dev/2023/03/OBDl2gw.webp)

`*` indica todos os e-mails, `i` é o prefixo do endereço de e-mail do usuário remetente criado acima. Para encaminhar e-mails, cada usuário precisa adicionar uma linha.

Em seguida, adicione o registro MX (aponto diretamente para o endereço do nome de domínio reverso aqui, conforme mostrado na primeira linha da figura abaixo).

![](https://pub-b8db533c86124200a9d799bf3ba88099.r2.dev/2023/03/7__KrU8.webp)

Após a conclusão da configuração, você pode usar outros endereços de e-mail para enviar e-mails para `i@wac.tax` e `any123@wac.tax` para ver se pode receber e-mails no Gmail.

Caso contrário, verifique o log chasquid ( `grep chasquid /var/log/syslog` ).

## Envie um e-mail para i@wac.tax com o Google Mail

Depois que o Google Mail recebeu o e-mail, naturalmente esperava responder com `i@wac.tax` em vez de i.wac.tax@gmail.com.

Visite [https://mail.google.com/mail/u/1/#settings/accounts](https://mail.google.com/mail/u/1/#settings/accounts) e clique em "Adicionar outro endereço de e-mail".

![](https://pub-b8db533c86124200a9d799bf3ba88099.r2.dev/2023/03/PAvyE3C.webp)

![](https://pub-b8db533c86124200a9d799bf3ba88099.r2.dev/2023/03/_OgLsPT.webp)

![](https://pub-b8db533c86124200a9d799bf3ba88099.r2.dev/2023/03/XIUf6Dc.webp)

Em seguida, digite o código de verificação recebido pelo e-mail para o qual foi encaminhado.

Por fim, pode ser definido como o endereço do remetente padrão (juntamente com a opção de responder com o mesmo endereço).

![](https://pub-b8db533c86124200a9d799bf3ba88099.r2.dev/2023/03/a95dO60.webp)

Desta forma, concluímos o estabelecimento do servidor de e-mail SMTP e, ao mesmo tempo, usamos o Google Mail para enviar e receber e-mails.

## Envie um e-mail de teste para verificar se a configuração foi bem-sucedida

Digite `ops/chasquid`

Execute `direnv allow` para instalar dependências (o direnv foi instalado no processo anterior de inicialização de uma chave e um gancho foi adicionado ao shell)

então corra

```
user=i@wac.tax pass=xxxx to=iuser.link@gmail.com ./sendmail.coffee
```

O significado dos parâmetros é o seguinte

* usuário: nome de usuário SMTP
* passar: senha SMTP
* para: destinatário

Você pode enviar um e-mail de teste.

![](https://pub-b8db533c86124200a9d799bf3ba88099.r2.dev/2023/03/ae1iWyM.webp)

Recomenda-se usar o Gmail para receber e-mails de teste para verificar se as configurações foram bem-sucedidas.

### Criptografia padrão TLS

Conforme a figura abaixo, existe este pequeno cadeado, o que significa que o certificado SSL foi habilitado com sucesso.

![](https://pub-b8db533c86124200a9d799bf3ba88099.r2.dev/2023/03/SrdbAwh.webp)

Em seguida, clique em "Mostrar e-mail original"

![](https://pub-b8db533c86124200a9d799bf3ba88099.r2.dev/2023/03/qQQsdxg.webp)

### DKIM

Conforme mostrado na figura abaixo, a página de e-mail original do Gmail exibe DKIM, o que significa que a configuração do DKIM foi bem-sucedida.

![](https://pub-b8db533c86124200a9d799bf3ba88099.r2.dev/2023/03/an6aXK6.webp)

Verifique o Recebido no cabeçalho do email original e você verá que o endereço do remetente é IPV6, o que significa que o IPV6 também foi configurado com sucesso.
