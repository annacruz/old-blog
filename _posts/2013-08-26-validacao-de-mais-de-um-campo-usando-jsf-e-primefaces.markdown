---
layout: post
title: "Validação de mais de um campo usando JSF e Primefaces"
date: 2013-08-26 11:11
comments: true
categories: [java, jsf, programação, validação]
---

Não sei se todos que me conhecem sabem, mas eu atualmente trabalho programando em Java. Particularmente esta não é a minha linguagem favorita, mas tem seu uso.

Bom, como já diz o título vou falar aqui de como fazer uma validação de mais de um campo tendo como condicional um terceiro campo usando JSF e PrimeFaces. Para isso vou fazer um simples validator do java. Bom mãos a obra?

<!-- more -->
No nosso formulário de contato existem três caixas de selectonemenu sendo respectivamente para país, estado e cidade. A validação é ser feita é que se o país for Brasil os campos de estado e cidade devem ser obrigatórios, ou seja, múltiplos campos são necessários para a nossa validação.

Originalmente vamos ter o nosso formulário deste jeito:

*código 1: form_contato.xhtml*

{% highlight java linenos %}
<h:form>
  <!-- Parte do código foi suprimida apenas deixamos os campos essenciais para o nosso exemplo-->
  <h:outputLabel value="País: " for="pais" />
  <p:selectOneMenu id="pais" value="#{contatoMB.idPais}" styleClass="label_280" required="true">
    <f:selectItems id="listaPais" value="#{contatoMB.listPaises}" var="pais"
      itemLabel="#{pais.nome}" itemValue="#{pais.id}" />
    <p:ajax update="uf cidade" listener="#{contatoMB.changePais()}" />
  </p:selectOneMenu>
  <p:message for="pais" />

  <h:outputLabel value="Estado: " for="uf" />
  <p:selectOneMenu id="uf" value="#{contatoMB.ufSigla}" required="true" disabled="#{not contatoMB.isBrasil()"}>
    <f:selectItem itemLabel="Selecione a UF" itemValue="" />
    <f:selectItems id="listaUfs" value="#{contatoMB.listUFs}" var="uf" itemLabel="#{uf.nome}"
      itemValue="#{uf.sigla}" />
    <p:ajax update="cidade" listener="#{contatoMB.changeUF()}" />
  </p:selectOneMenu>
  <p:message for="uf" />

  <h:outputLabel value="Cidade" for="cidade" />
  <p:selectOneMenu id="cidade" value="#{contatoMB.idCidade}" required="true" disabled="#{not contatoMB.isBrasil()"}>
    <f:selectItem itemLabel="Selecione a Cidade" itemValue="" />
    <f:selectItems id="listaCidades" value="#{contatoMB.listCidades}" var="cidade"
      itemLabel="#{cidade.nome}" itemValue="#{cidade.id}" />
  </p:selectOneMenu>
  <p:message for="cidade" />
</h:form>
{% endhighlight %}

OBS: Não é o objetivo aqui explicar como fazer o ajax para ao mudar de país habilitar ou desabilitar a validação de estado e cidade, nem também o carregamento das cidades ao trocar de estado. Não é muito difícil fazer isso e caso seja necessário eu faço um post sobre isso dps =)

Para utilizarmos o validator primeiramente vamos fazer um binding dos campos de país, estado e cidade para atributos que ficam incluidos dentro de um input hidden isso tudo no próprio formulário. Isto se faz necessário para que o validator consiga pegar o componente completo, e assim, consigamos deixar o campo destacado em vermelho após a validação. Dentro desse input hidden também vamos apontar qual o nome do nosso validador. Com estas modificações o nosso xhtml vai ficar assim:

*código 2: form_contato.xhtml*

{% highlight java linenos %}
<h:form>
  <!-- Parte do código foi suprimida apenas deixamos os campos essenciais para o nosso exemplo-->

  <h:inputHidden value="true" id="camposValidacao">
    <f:validator validatorId="paisEstadoCidadeValidator" />
    <f:attribute name="paisVal" value="#{paisVal}" />
    <f:attribute name="estadoVal" value="#{estadoVal}" />
    <f:attribute name="cidadeVal" value="#{cidadeVal}" />
  </h:inputHidden>

  <h:outputLabel value="País: " for="pais" />
  <p:selectOneMenu id="pais" value="#{contatoMB.idPais}" styleClass="label_280" required="true" binding="{paisVal}">
    <f:selectItems id="listaPais" value="#{contatoMB.listPaises}" var="pais"
      itemLabel="#{pais.nome}" itemValue="#{pais.id}" />
    <p:ajax update="uf cidade" listener="#{contatoMB.changePais()}" />
  </p:selectOneMenu>
  <p:message for="pais" />

  <h:outputLabel value="Estado: " for="uf" />
  <p:selectOneMenu id="uf" value="#{contatoMB.ufSigla}" required="true" disabled="#{not contatoMB.isBrasil()"} binding="{estadoVal}">
    <f:selectItem itemLabel="Selecione a UF" itemValue="" />
    <f:selectItems id="listaUfs" value="#{contatoMB.listUFs}" var="uf" itemLabel="#{uf.nome}"
      itemValue="#{uf.sigla}" />
    <p:ajax update="cidade" listener="#{contatoMB.changeUF()}" />
  </p:selectOneMenu>
  <p:message for="uf" />

  <h:outputLabel value="Cidade" for="cidade" />
  <p:selectOneMenu id="cidade" value="#{contatoMB.idCidade}" required="true" disabled="#{not contatoMB.isBrasil()"} binding="{cidadeVal}">
    <f:selectItem itemLabel="Selecione a Cidade" itemValue="" />
    <f:selectItems id="listaCidades" value="#{contatoMB.listCidades}" var="cidade"
      itemLabel="#{cidade.nome}" itemValue="#{cidade.id}" />
  </p:selectOneMenu>
  <p:message for="cidade" />
</h:form>
{% endhighlight %}

Jóia! Agora vamos para a parte divertida: fazer o validador. Nossa classe validadora vai simplismente implementar a interface Validator do javax.faces e incluir os nossos métodos de validação. Ela vai ficar assim:

*código 3: PaisCidadeEstadoValidator.java*

{% highlight java linenos %}

// Os imports foram suprimidos deixando apenas o principal da classe.
// É importante fazer a anotação do FacesValidator pois é apartir dela que o xhtml reconhece quem é o validator a ser utilizado.

@FacesValidator(value = "paisEstadoCidadeValidator")
public class PaisEstadoCidadeValidator implements Validator {

    // Injeção do ResourceBundle para que possamos colocar a mensagem de validação dentro do <p:message>
    @Inject
    @Name(value = "ValidationMessages")
    ResourceBundle validationBundle;

    // Esta é a função que faz a validação de fato.
    @Override
    public void validate(FacesContext context, UIComponent component, Object value) throws ValidatorException {

        // Primeiro eu pego todos os componentes que estão no xhtml, note que eu pego eles através do seu id que foi
        // referenciado dentro do atributo incluido no input hidden.
        UIInput inputPais   = (UIInput) component.getAttributes().get("paisVal");
        UIInput inputEstado = (UIInput) component.getAttributes().get("estadoVal");
        UIInput inputCidade = (UIInput) component.getAttributes().get("cidadeVal");

        // Aqui está um detalhe importante: se não vier nada dentro dos atributos correspondentes a estado e cidade
        // é sinal de que os campos estavam desabilitados em tela, portanto, o país não é Brasil. Cabe lembrar aqui que
        // não faz parte do escopo deste post como foi feita essa validação do país ser ou não Brasil.
        if (inputEstado.getSubmittedValue() == null|| inputCidade.getSubmittedValue() == null){
            return;
        }

        // Agora é só pegar os atributos para podermos fazer as validações em si.
        String pais   = inputPais.getSubmittedValue().toString();
        String estado = inputEstado.getSubmittedValue().toString();
        String cidade = inputCidade.getSubmittedValue().toString();

        // Variáveis que sempre permanecem as mesmas e servem para o lançamento das mensagens de erro.
        String errorMsg = "Campo obrigatório.";
        FacesMessage msg = new FacesMessage(FacesMessage.SEVERITY_ERROR, errorMsg, errorMsg);

        // Aqui está a validação em si
        if (pais.equals("Brasil")) {
            if (estado.isEmpty()) {
                inputEstado.setValid(false);
                context.addMessage(inputEstado.getClientId(), msg);
            }
            if (cidade.isEmpty()) {
                inputCidade.setValid(false);
                context.addMessage(inputCidade.getClientId(), msg);
            }
        }
    }
}
{% endhighlight  %}

Pronto acabou! Sim é só isso que você precisa pra poder fazer essa validação combinada com mais de um campo =)

Viu como é super simples no fim das contas? No próximo post vou fazer a validação a partir de um evento ajax.
