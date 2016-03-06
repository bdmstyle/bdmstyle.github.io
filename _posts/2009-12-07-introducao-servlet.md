---
layout: post
title: Funcionamento de uma webapp em java sem IDEs 
date: 2009-12-07 00:11:36.000000000 -02:00
type: post
published: true
status: publish
tags: [webapps]
---
Muita gente saber criar um `Dynamic Web Project` no eclipse, mas não sabe realmente como funciona uma aplicação web em java.

## O que é uma aplicação web?

Uma aplicação web indepentente de ser escrita em java é composta de  ao menos dois items importantes:

* O cliente é um navegador como o firefox, ele faz requisições ao servidor pedindo paginas.
* O servidor é um software que normalmente roda em um servidor na internet para responder requisições de clientes.

Dado esses items vamos a um exemplo de comunicação simples entre eles.

1. Você acessa uma pagina por exemplo `www.sample.com`, o navegador (cliente) faz uma requisição ao servidor pedindo uma pagina, por exemplo `sample.html`.

2. O servidor procura a pagina `sample.html` em seu hd e entrega o arquivo para o navegador , assim finalmente a pagina é exibida.

Um servidor como o `Apache` ou `Nginx` pode fornecer esse serviço de procurar arquivos de forma `estática` (**por exemplo html, css e javascript**)  dentro do servidor e retornar ao cliente, neste caso não é nescessario nenhuma programação do lado do servidor, afinal ele serve apenas arquivos estaticos (**que já existem no hd do servidor**).

Mas e se desejarmos que a pagina tivesse algum conteúdo dinamico, ou seja algo que não existe no hd do servidor e que é gerado na hora exclusivamente para de quem fez a requisição naquele instante, por exemplo a **"hora atual do servidor + 3 horas de fuso"**. 

Neste caso um servidor que apenas serve coisas prontas como o `Apache` não ajuda, e precisamos de uma aplicação web para dar essa dinamica.

Uma aplicação web é um conjunto de arquivos java com sua logica de negócio compactados no formato `war` ou em forma de pasta descompactada, seguindo a estrutura de diretorios e arquivos da especificação **Servlet**, que diz como tem que ser estruturadas as aplicações web em java. 

Por sua vez esse arquivo war é executado em um  **servidor web** ou **web container** que também segue a especificação de **Servlet** mas neste caso para servidores, ou seja ele sabe executar aplicações estruturadas no formato war.


## Criando sua aplicação web

Nossa webapp tem o nome de `sample` vamos criar uma pasta com esse nome no diretório `home` que será a raiz da webapp, portanto vamos acessar a webapp a partir de http://localhost:8080/sample/ 
	
	sample
	|  static.html (Conteúdo estático)
	|  dynamic.jsp (Contúdo Dinamico)
	|
	|__WEB-INF
	   |___classes
	       |  MyServlet.class (Servlet compilado)

Dentro de sample podemos colocar arquivos estáticos como o `static.html`, que ficam acessiveis diretamente via http://localhost:8080/sample/static.html

{% highlight html %}
<html>
	<body>
	static file 
	</body>
</html> 
{% endhighlight %}

Vamos criar nosso Servlet, a classe que gera conteudo dinamico no servidor. Crie `MyServlet.java`  no seu diretorio "home" (que é a pasta de seu usuário), `~` no linux.

{% highlight java %}
import java.io.IOException;
import javax.servlet.RequestDispatcher;
import javax.servlet.ServletException;
import javax.servlet.annotation.WebServlet;
import javax.servlet.http.HttpServlet;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;


@WebServlet("/dynamic")
public class MyServlet extends HttpServlet {

  @Override
  protected void doGet(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
  RequestDispatcher rd = request.getRequestDispatcher("/dynamic.jsp");
  request.setAttribute("name","dynamic");
  rd.forward(request, response);
 }
}
{% endhighlight %}

No servlet em `@WebServlet("/dynamic")` indicamos que quando alguem chamar http://localhost:8080/sample/dynamic o MyServlet torna disponível a variavel `name` com o conteúdo "dynamic" para o jsp e em seguida exibe a pagina `dynamic.jsp`

Agora vamos ao conteúdo do `dynamic.jsp` (Java Server Page) repare na variável `${name}` quem vem com o valor dinamico do servidor sendo acessado pela EL (Expression Laguage) no jsp.

{% highlight jsp %}
<html>
	<body>
	Hi ${name}
	</body>
</html>
{% endhighlight %}

##Instalando servidor

Vamos usar o `Apache Tomcat` que implementa a especificação de **Servlet** e tem a capacidade de servir tanto arquivos estaticos quanto gerar arquivos dinamicos.

O pré requisito é ter o java instalado corretamente, para saber se está tudo certo abra um terminal e digite `java -version`. Se o comando mostrar a versão do java tudo ok, agora 
vamos instalar o `Apache Tomcat` para servir nossas paginas. Vá em <http://tomcat.apache.org/> na parte de downloads e baixe a última versão no formato **zip**. Descompacte
 o arquivo no seu diretório `home` e renomeie para `tomcat` a pasta, os diretórios mais importantes são: 
 
 * `tomcat\bin`, essa pasta tem os arquivos `startup` e `shutdown` (no linux tem que dar permissão) para ligar e desligar o servidor

 * `tomcat\webapps`, essa pasta é onde você deve colocar suas aplicações web.
 
 * `tomcat\logs`, essa pasta é onde você pode ver o log do servidor.

##Instalando webapp no servidor

Para instalar nossa webapp no servidor (realizar o deploy), temos que copiar a pasta sample dentro de webapps

           tomcat
     	webpps	
		sample
		|  static.html (Conteúdo estático)
		|  dynamic.jsp (Contúdo Dinamico)
		|
		|__WEB-INF
	   	|___classes

Para finalizar tempos que compilar o arquivo `MyServlet.java` que vai gerar MyServlet.class e colocar ele na pasta classes da estrutura da webapp.
Dado que tanto o apache quanto o arquivo `MyServlet.java` estão no seu diretorio "home" (que é a pasta de seu usuário) `~` no linux.

Rodamos:
`javac -cp ~/apache/lib/servlet-api.jar ~/MyServlet.java -d ~/apache/webapps/sample/WEB-INF/classes/`
Esse comando compila o servlet e coloca o arquivo gerado MyServlet.class na pasta classes dentro de WEB-INF ficando a assim:

Todo servlet é filho de HttpServlet, as bibliotecas para fazer webapps não vem por padrão com a instalação do java, afinal poderia ser um programa para celular ou desktop, mas quando instalamos o tomcat na pasta lib, ele ja tem a biblioteca chamada servlet-api.jar que tem todas as classes base para se fazer um servlet.


	tomcat
	webapps
	sample
	|  static.html (Conteúdo estático)
	|  dynamic.jsp (Contúdo Dinamico)
	|
	|__WEB-INF
	   |___classes
	       |  MyServlet.class (Servlet compilado)

Todo servlet tem que ser "Filho" da classe mae servlet.. mais onde fica esta classe mae de servlet? quando voce instalou o tomcat,na pasta lib tem uma biblioteca chamada servlet-api.jar, esse arquivo tem todas as classes base para se fazer um servlet.

## Rodando sua aplicação web java

Entre em `tomcat\bin`, e rode o arquivo `statup` assim o servidor vai subir, disponibilizando a webapp.





