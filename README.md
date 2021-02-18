# Tomcat-6-SSL-Configuration-HOW-TO
:computer: # COMO FAZER Configuração SSL

A descrição abaixo usa o nome da variável $ CATALINA_BASE para se referir ao diretório base no qual a maioria dos caminhos relativos são resolvidos. Se você não configurou o Tomcat 6 para instâncias múltiplas definindo um diretório CATALINA_BASE, então $ CATALINA_BASE será definido com o valor de $ CATALINA_HOME, o diretório no qual você instalou o Tomcat 6.

Para instalar e configurar o suporte SSL no Tomcat 6, você precisa seguir estas etapas simples. Para obter mais informações, leia o restante deste COMO FAZER.

Crie um arquivo de armazenamento de chaves para armazenar a chave privada do servidor e o certificado autoassinado executando o seguinte comando:
### Sistema Operacional Windows
```
"%JAVA_HOME%\bin\keytool" -genkey -alias tomcat -keyalg RSA
```
### Sistema Operacional Unix
```
"$JAVA_HOME/bin/keytool -genkey -alias tomcat -keyalg RSA
```
e especifique um valor de senha de "changeit".

Remova o comentário da entrada "Conector SSL HTTP / 1.1" em $ CATALINA_BASE / conf / server.xml e modifique conforme descrito na seção Configuração abaixo.

### Introdução ao SSL
SSL, ou Secure Socket Layer, é uma tecnologia que permite que navegadores e servidores da web se comuniquem por meio de uma conexão segura. Isso significa que os dados enviados são criptografados por um lado, transmitidos e então descriptografados pelo outro lado antes do processamento. Este é um processo bidirecional, o que significa que o servidor e o navegador criptografam todo o tráfego antes de enviar os dados.

Outro aspecto importante do protocolo SSL é a autenticação. Isso significa que durante sua tentativa inicial de se comunicar com um servidor web por meio de uma conexão segura, esse servidor apresentará ao seu navegador um conjunto de credenciais, na forma de um "Certificado", como prova de que o site é quem e o que afirma ser estar. Em certos casos, o servidor também pode solicitar um certificado de seu navegador da web, pedindo uma prova de que você é quem afirma ser. Isso é conhecido como "Autenticação de cliente", embora na prática seja mais usado para transações business-to-business (B2B) do que com usuários individuais. A maioria dos servidores da web habilitados para SSL não solicitam autenticação de cliente.

### SSL e Tomcat
É importante observar que configurar o Tomcat para aproveitar as vantagens de soquetes seguros geralmente é necessário apenas ao executá-lo como um servidor da web autônomo. Ao executar o Tomcat principalmente como um contêiner Servlet / JSP atrás de outro servidor da web, como Apache ou Microsoft IIS, geralmente é necessário configurar o servidor da web primário para lidar com as conexões SSL dos usuários. Normalmente, este servidor negociará todas as funcionalidades relacionadas ao SSL e, em seguida, transmitirá quaisquer solicitações destinadas ao contêiner Tomcat somente após descriptografar essas solicitações. Da mesma forma, o Tomcat retornará respostas em texto não criptografado, que serão criptografadas antes de serem devolvidas ao navegador do usuário. Nesse ambiente, o Tomcat sabe que as comunicações entre o servidor da web primário e o cliente estão ocorrendo por meio de uma conexão segura (porque seu aplicativo precisa ser capaz de perguntar sobre isso), mas não participa da criptografia ou descriptografia em si.

### Certificados
Para implementar SSL, um servidor web deve ter um certificado associado para cada interface externa (endereço IP) que aceita conexões seguras. A teoria por trás desse design é que um servidor deve fornecer algum tipo de garantia razoável de que seu proprietário é quem você pensa que é, especialmente antes de receber qualquer informação sensível. Embora uma explicação mais ampla dos certificados esteja além do escopo deste documento, pense em um certificado como um "passaporte digital" para um endereço de Internet. Ele declara a organização à qual o site está associado, junto com algumas informações básicas de contato sobre o proprietário ou administrador do site.

Este certificado é assinado criptograficamente por seu proprietário e, portanto, é extremamente difícil para outra pessoa falsificá-lo. Para que o certificado funcione nos navegadores dos visitantes sem avisos, ele precisa ser assinado por um terceiro confiável. Eles são chamados de Autoridades de Certificação (CAs). Para obter um certificado assinado, você precisa escolher uma CA e seguir as instruções fornecidas por sua CA para obter seu certificado. Uma variedade de CAs está disponível, incluindo algumas que oferecem certificados gratuitamente.

Java fornece uma ferramenta de linha de comando relativamente simples, chamada keytool, que pode facilmente criar um certificado "autoassinado". Os certificados autoassinados são simplesmente certificados gerados pelo usuário que não foram assinados por uma CA bem conhecida e, portanto, não têm garantia de autenticidade alguma. Embora os certificados autoassinados possam ser úteis para alguns cenários de teste, eles não são adequados para qualquer forma de uso em produção.

### Dicas gerais para executar SSL
Ao proteger um site com SSL, é importante certificar-se de que todos os ativos que o site usa são servidos por SSL, para que um invasor não possa contornar a segurança injetando conteúdo malicioso em um arquivo javascript ou similar. Para aumentar ainda mais a segurança do seu site, você deve avaliar o uso do cabeçalho HSTS. Ele permite que você comunique ao navegador que seu site deve sempre ser acessado por https.

O uso de hosts virtuais baseados em nome em uma conexão segura requer configuração cuidadosa dos nomes especificados em um único certificado ou Tomcat 8.5 em diante, onde o suporte para indicação de nome de servidor (SNI) está disponível. O SNI permite que vários certificados com nomes diferentes sejam associados a um único conector TLS.

### Configuração
## Prepare o armazenamento de chaves de certificado
O Tomcat atualmente opera apenas em keystores de formato JKS, PKCS11 ou PKCS12. O formato JKS é o formato "Java KeyStore" padrão do Java e é o formato criado pelo utilitário de linha de comando keytool. Essa ferramenta está incluída no JDK. O formato PKCS12 é um padrão da Internet e pode ser manipulado via (entre outras coisas) OpenSSL e Key-Manager da Microsoft.

Cada entrada em um keystore é identificada por uma string de alias. Enquanto muitas implementações de keystore tratam aliases de uma maneira que não diferencia maiúsculas de minúsculas, implementações que diferenciam maiúsculas de minúsculas estão disponíveis. A especificação PKCS11, por exemplo, requer que os aliases façam distinção entre maiúsculas e minúsculas. Para evitar problemas relacionados à diferenciação de maiúsculas e minúsculas de aliases, não é recomendado usar aliases que diferem apenas em maiúsculas e minúsculas.

Para importar um certificado existente para um keystore JKS, leia a documentação (em seu pacote de documentação JDK) sobre o keytool. Observe que o OpenSSL geralmente adiciona comentários legíveis antes da chave, mas o keytool não oferece suporte para isso. Portanto, se o seu certificado tiver comentários antes dos dados da chave, remova-os antes de importar o certificado com o keytool.

Para importar um certificado existente assinado por sua própria CA em um keystore PKCS12 usando OpenSSL, você executaria um comando como:
```
openssl pkcs12 -export -in mycert.crt -inkey mykey.key
                        -out mycert.p12 -name tomcat -CAfile myCA.crt
                        -caname root -chain
```

Para criar um novo keystore JKS do zero, contendo um único certificado autoassinado, execute o seguinte em uma linha de comando de terminal:

### Windows		
```
"%JAVA_HOME%\bin\keytool" -genkey -alias tomcat -keyalg RSA
```
### Unix		
```	
$JAVA_HOME/bin/keytool -genkey -alias tomcat -keyalg RSA
```
(O algoritmo RSA deve ser preferido como um algoritmo seguro, e isso também garante compatibilidade geral com outros servidores e componentes.)

Este comando criará um novo arquivo, no diretório inicial do usuário sob o qual você o executa, denominado ".keystore". Para especificar um local ou nome de arquivo diferente, inclua o parâmetro -keystore, seguido do nome do caminho completo para seu arquivo de armazenamento de chave, ao comando keytool mostrado acima. Você também precisará refletir esse novo local no arquivo de configuração server.xml, conforme descrito posteriormente. Por exemplo:

### Windows
```
"%JAVA_HOME%\bin\keytool" -genkey -alias tomcat -keyalg RSA
  -keystore \path\to\my\keystore
  ```
### Unix
```
$JAVA_HOME/bin/keytool -genkey -alias tomcat -keyalg RSA
  -keystore /path/to/my/keystore
```
Depois de executar esse comando, primeiro será solicitada a senha do keystore. A senha padrão usada pelo Tomcat é "changeit" (todas em minúsculas), embora você possa especificar uma senha personalizada se desejar. Você também precisará especificar a senha personalizada no arquivo de configuração server.xml, conforme descrito posteriormente.

Em seguida, serão solicitadas informações gerais sobre este certificado, como empresa, nome de contato e assim por diante. Essas informações serão exibidas para usuários que tentarem acessar uma página segura em seu aplicativo, portanto, certifique-se de que as informações fornecidas aqui correspondam ao que eles esperam.

Por fim, será solicitada a senha da chave, que é a senha especificamente para este certificado (ao contrário de quaisquer outros certificados armazenados no mesmo arquivo de armazenamento de chaves). Você DEVE usar a mesma senha aqui que foi usada para a própria senha do keystore. Esta é uma restrição da implementação do Tomcat. (Atualmente, o prompt do keytool dirá que pressionar a tecla ENTER faz isso para você automaticamente.)

Se tudo deu certo, agora você tem um arquivo de armazenamento de chave com um certificado que pode ser usado por seu servidor.

### Nota
A senha da chave privada e a senha do keystore devem ser iguais. Se eles forem diferentes, você receberá um erro nas linhas de java.io.IOException: Não é possível recuperar a chave, conforme documentado no problema 38217 do Bugzilla (https://bz.apache.org/bugzilla/show_bug.cgi?id=38217), que contém referências adicionais para esse problema.

### Edite o arquivo de configuração do Tomcat

O Tomcat pode usar duas implementações diferentes de SSL:

- a implementação JSSE fornecida como parte do Java runtime (desde 1.4)
- a implementação APR, que usa o mecanismo OpenSSL por padrão.
Os detalhes exatos da configuração dependem de qual implementação está sendo usada. Se você configurou o Conector especificando o protocolo genérico = "HTTP / 1.1", a implementação usada pelo Tomcat é escolhida automaticamente. Se a instalação usar APR - ou seja, você instalou a biblioteca nativa do Tomcat - então ela usará a implementação SSL APR, caso contrário, usará a implementação Java JSSE.
Como os atributos de configuração para suporte SSL diferem significativamente entre as implementações APR vs. JSSE, é recomendado evitar a seleção automática da implementação. Isso é feito especificando um nome de classe no atributo de protocolo do Conector.

Para definir um conector Java (JSSE), independentemente de a biblioteca APR estar carregada ou não, use um dos seguintes:

```	
<!-- Define a HTTP/1.1 Connector on port 8443, JSSE BIO implementation -->
<Connector protocol="org.apache.coyote.http11.Http11Protocol"
           port="8443" .../>

<!-- Define a HTTP/1.1 Connector on port 8443, JSSE NIO implementation -->
<Connector protocol="org.apache.coyote.http11.Http11NioProtocol"
           port="8443" .../>
```
Como alternativa, para especificar um conector APR (a biblioteca APR deve estar disponível), use:

```
<!-- Define a HTTP/1.1 Connector on port 8443, APR implementation -->
<Connector protocol="org.apache.coyote.http11.Http11AprProtocol"
           port="8443" .../>
```
Se você estiver usando o APR, terá a opção de configurar um mecanismo alternativo para OpenSSL.

```
<Listener className="org.apache.catalina.core.AprLifecycleListener"
          SSLEngine="someengine" SSLRandomSeed="somedevice" />
```
o valor padrão é:

```
<Listener className="org.apache.catalina.core.AprLifecycleListener"
          SSLEngine="on" SSLRandomSeed="builtin" />
```

Portanto, para usar SSL no APR, certifique-se de que o atributo SSLEngine esteja definido como algo diferente de desligado. O valor padrão é on e se você especificar outro valor, ele deve ser um nome de mecanismo válido.
SSLRandomSeed permite especificar uma fonte de entropia. O sistema produtivo precisa de uma fonte confiável de entropia, mas a entropia pode precisar de muito tempo para ser coletada, portanto, os sistemas de teste não podem usar fontes de entropia bloqueadoras como "/ dev / urandom", o que permitirá um início mais rápido do Tomcat.

A etapa final é configurar o Conector no arquivo $ CATALINA_BASE / conf / server.xml, onde $ CATALINA_BASE representa o diretório base para a instância do Tomcat 6. Um exemplo de elemento <Connector> para um conector SSL está incluído no arquivo server.xml padrão instalado com Tomcat. Para JSSE, deve ser parecido com isto:
  
```
<!-- Define a SSL Coyote HTTP/1.1 Connector on port 8443 -->
<Connector
           protocol="org.apache.coyote.http11.Http11Protocol"
           port="8443" maxThreads="200"
           scheme="https" secure="true" SSLEnabled="true"
           keystoreFile="${user.home}/.keystore" keystorePass="changeit"
           clientAuth="false" sslProtocol="TLS"/>
```

O conector APR usa atributos diferentes para muitas configurações SSL, principalmente chaves e certificados. Um exemplo de configuração APR é:

```
	
	
<!-- Define a SSL Coyote HTTP/1.1 Connector on port 8443 -->
<Connector
           protocol="org.apache.coyote.http11.Http11AprProtocol"
           port="8443" maxThreads="200"
           scheme="https" secure="true" SSLEnabled="true"
           SSLCertificateFile="/usr/local/ssl/server.crt" 
           SSLCertificateKeyFile="/usr/local/ssl/server.pem"
           SSLVerifyClient="optional" SSLProtocol="TLSv1+TLSv1.1+TLSv1.2"/>
```
As opções de configuração e as informações sobre quais atributos são obrigatórios para os conectores baseados em JSSE (BIO e NIO) estão documentadas na seção Suporte SSL da referência de configuração do conector HTTP. As opções de configuração e as informações sobre quais atributos são obrigatórios para o conector APR estão documentadas na seção HTTPS do APR How-To.

O atributo port é o número da porta TCP / IP na qual o Tomcat escutará conexões seguras. Você pode alterá-lo para qualquer número de porta que desejar (como a porta padrão para comunicações https, que é 443). No entanto, uma configuração especial (fora do escopo deste documento) é necessária para executar o Tomcat em números de porta inferiores a 1024 em muitos sistemas operacionais.

```
Se você alterar o número da porta aqui, também deverá alterar o valor especificado para o atributo redirectPort no conector não SSL. Isso permite que o Tomcat redirecione automaticamente os usuários que tentam acessar uma página com uma restrição de segurança especificando que SSL é necessário, conforme exigido pela Especificação de Servlet.
```

Depois de concluir essas mudanças de configuração, você deve reiniciar o Tomcat como normalmente faz e deve estar no negócio. Você deve conseguir acessar qualquer aplicativo da web compatível com o Tomcat via SSL. Por exemplo, tente:

```
https://localhost:8443/
```

e você deverá ver a página inicial normal do Tomcat (a menos que tenha modificado o aplicativo da web ROOT). Se isso não funcionar, a seção a seguir contém algumas dicas de solução de problemas.

### Instalando um certificado de uma autoridade de certificação
Para obter e instalar um certificado de uma autoridade de certificação (como verisign.com, thawte.com ou trustcenter.de), leia a seção anterior e siga estas instruções:

### Criar uma Solicitação de Assinatura de Certificado (CSR) local
Para obter um certificado da autoridade de certificação de sua escolha, você deve criar uma chamada Solicitação de assinatura de certificado (CSR). Esse CSR será usado pela autoridade de certificação para criar um certificado que identificará o seu site como "seguro". Para criar um CSR, siga estas etapas:

- Crie um certificado autoassinado local (conforme descrito na seção anterior):

```
keytool -genkey -alias tomcat -keyalg RSA
    -keystore <your_keystore_filename>
```

Observação: em alguns casos, você terá que inserir o domínio do seu site (ou seja, www.myside.org) no campo "nome e sobrenome" para criar um certificado funcional.
- O CSR é então criado com:

```
keytool -certreq -keyalg RSA -alias tomcat -file certreq.csr
    -keystore <your_keystore_filename>
```

Agora você tem um arquivo chamado certreq.csr que pode enviar para a Autoridade de Certificação (veja a documentação do site da Autoridade de Certificação para saber como fazer isso). Em troca, você recebe um certificado.

### Importando o Certificado
Agora que você tem seu certificado, pode importá-lo para o armazenamento de chaves local. Em primeiro lugar, você deve importar um certificado de cadeia ou certificado raiz para o seu armazenamento de chaves. Depois disso, você pode prosseguir com a importação do seu certificado.

- Baixe um certificado de cadeia da autoridade de certificação da qual você obteve o certificado.
Para certificados comerciais da Verisign.com, vá para: http://www.verisign.com/support/install/intermediate.html
Para obter os certificados de avaliação da Verisign.com, acesse: http://www.verisign.com/support/verisign-intermediate-ca/Trial_Secure_Server_Root/index.html
Para Trustcenter.de vá para: http://www.trustcenter.de/certservices/cacerts/en/en.htm#server
Para Thawte.com vá para: http://www.thawte.com/certs/trustmap.html
- Importe o certificado da cadeia em seu armazenamento de chaves

```
keytool -import -alias root -keystore <your_keystore_filename>
    -trustcacerts -file <filename_of_the_chain_certificate>
```

E finalmente importe seu novo certificado

```
keytool -import -alias tomcat -keystore <your_keystore_filename>
    -file <your_certificate_filename>
```

### Solução de problemas
Aqui está uma lista de problemas comuns que você pode encontrar ao configurar comunicações SSL e o que fazer a respeito.

- Quando o Tomcat é inicializado, recebo uma exceção como "java.io.FileNotFoundException: {algum-diretório} / {algum-arquivo} não encontrado".
Uma explicação provável é que o Tomcat não consegue encontrar o arquivo de armazenamento de chave onde está procurando. Por padrão, o Tomcat espera que o arquivo keystore seja denominado .keystore no diretório inicial do usuário sob o qual o Tomcat está sendo executado (que pode ou não ser igual ao seu :-). Se o arquivo keystore estiver em qualquer outro lugar, você precisará adicionar um atributo keystoreFile ao elemento <Connector> no arquivo de configuração do Tomcat.

- Quando o Tomcat é inicializado, recebo uma exceção como "java.io.FileNotFoundException: Keystore foi adulterado ou a senha estava incorreta".
Supondo que alguém não tenha realmente adulterado seu arquivo de armazenamento de chave, a causa mais provável é que o Tomcat está usando uma senha diferente da que você usou quando criou o arquivo de armazenamento de chave. Para corrigir isso, você pode voltar e recriar o arquivo keystore ou pode adicionar ou atualizar o atributo keystorePass no elemento <Connector> no arquivo de configuração Tomcat. LEMBRETE - As senhas diferenciam maiúsculas de minúsculas!
	
- Quando o Tomcat é inicializado, recebo uma exceção como "java.net.SocketException: erro de handshake SSL javax.net.ssl.SSLException: Nenhum certificado ou chave disponível corresponde aos conjuntos de criptografia SSL ativados."

Uma explicação provável é que o Tomcat não pode localizar o alias da chave do servidor no armazenamento de chaves especificado. Verifique se o keystoreFile e keyAlias corretos estão especificados no elemento <Connector> no arquivo de configuração do Tomcat. LEMBRETE - os valores keyAlias podem fazer distinção entre maiúsculas e minúsculas!

Se você ainda estiver tendo problemas, uma boa fonte de informações é a lista de discussão TOMCAT-USER. Você pode encontrar dicas para arquivos de mensagens anteriores nesta lista, bem como informações de inscrição e cancelamento de inscrição, em http://tomcat.apache.org/lists.html

Dicas e bits diversos
Para acessar o ID da sessão SSL da solicitação, use:
String sslID = (String) request.getAttribute ("javax.servlet.request.ssl_session");
Para uma discussão adicional sobre esta área, consulte https://bz.apache.org/bugzilla/show_bug.cgi?id=22679
