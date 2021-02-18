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
SSL, ou Secure Socket Layer, é uma tecnologia que permite que navegadores e servidores da web se comuniquem por meio de uma conexão segura. Isso significa que os dados enviados são criptografados por um lado, transmitidos e então descriptografados pelo outro lado antes do processamento. Este é um processo bidirecional, o que significa que o servidor E o navegador criptografam todo o tráfego antes de enviar os dados.

Outro aspecto importante do protocolo SSL é a autenticação. Isso significa que durante sua tentativa inicial de se comunicar com um servidor web por meio de uma conexão segura, esse servidor apresentará ao seu navegador um conjunto de credenciais, na forma de um "Certificado", como prova de que o site é quem e o que afirma ser estar. Em certos casos, o servidor também pode solicitar um certificado de seu navegador da web, pedindo uma prova de que você é quem afirma ser. Isso é conhecido como "Autenticação de cliente", embora na prática seja mais usado para transações business-to-business (B2B) do que com usuários individuais. A maioria dos servidores da web habilitados para SSL não solicitam autenticação de cliente.

### SSL e Tomcat
É importante observar que configurar o Tomcat para aproveitar as vantagens de soquetes seguros geralmente é necessário apenas ao executá-lo como um servidor da web autônomo. Ao executar o Tomcat principalmente como um contêiner Servlet / JSP atrás de outro servidor da web, como Apache ou Microsoft IIS, geralmente é necessário configurar o servidor da web primário para lidar com as conexões SSL dos usuários. Normalmente, este servidor negociará todas as funcionalidades relacionadas ao SSL e, em seguida, transmitirá quaisquer solicitações destinadas ao contêiner Tomcat somente após descriptografar essas solicitações. Da mesma forma, o Tomcat retornará respostas em texto não criptografado, que serão criptografadas antes de serem devolvidas ao navegador do usuário. Nesse ambiente, o Tomcat sabe que as comunicações entre o servidor da web primário e o cliente estão ocorrendo por meio de uma conexão segura (porque seu aplicativo precisa ser capaz de perguntar sobre isso), mas não participa da criptografia ou descriptografia em si.

### Certificados
Para implementar SSL, um servidor web deve ter um certificado associado para cada interface externa (endereço IP) que aceita conexões seguras. A teoria por trás desse design é que um servidor deve fornecer algum tipo de garantia razoável de que seu proprietário é quem você pensa que é, especialmente antes de receber qualquer informação sensível. Embora uma explicação mais ampla dos certificados esteja além do escopo deste documento, pense em um certificado como um "passaporte digital" para um endereço de Internet. Ele declara a organização à qual o site está associado, junto com algumas informações básicas de contato sobre o proprietário ou administrador do site.

Este certificado é assinado criptograficamente por seu proprietário e, portanto, é extremamente difícil para outra pessoa falsificá-lo. Para que o certificado funcione nos navegadores dos visitantes sem avisos, ele precisa ser assinado por um terceiro confiável. Eles são chamados de Autoridades de Certificação (CAs). Para obter um certificado assinado, você precisa escolher uma CA e seguir as instruções fornecidas por sua CA para obter seu certificado. Uma variedade de CAs está disponível, incluindo algumas que oferecem certificados gratuitamente.

Java fornece uma ferramenta de linha de comando relativamente simples, chamada keytool, que pode facilmente criar um certificado "autoassinado". Os certificados autoassinados são simplesmente certificados gerados pelo usuário que não foram assinados por uma CA bem conhecida e, portanto, não têm garantia de autenticidade alguma. Embora os certificados autoassinados possam ser úteis para alguns cenários de teste, eles não são adequados para qualquer forma de uso em produção.
