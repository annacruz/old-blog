---
layout: post
title: "Instalando token PCKS#11 no mac"
date: 2014-03-18 19:09:41 -0300
comments: true
categories:  [token, e-cpf, certificado digital, firefox, pcks#11]
---

Essa semana eu me deparei com um pequeno problema: instalar meu e-cpf (certificado digital pessoal) no meu macbook, no meu caso ele é um token usb do tipo PCKS#11. Nesse post eu vou tentar explicar como
conseguir fazer ele funcionar direitinho e bem rápido.
<!-- more -->

Um e-cpf para quem não sabe é um token usb ou smartcard que contém informações pessoais suas criptorgrafadas e serve para assinar digitalmente documentos
além de poder ser usado para assinar a declaração do imposto de renda e recuperar informações de declarações antigas no site da receita federal. Para conseguir
o seu, você tem que solicitar na [Receita Federal](http://www.receita.fazenda.gov.br/atendvirtual/informacoesbasicas/comoobter.htm) e pagar uma taxa.

A instalação do e-cpf começa com a instalação do driver do token, em geral esse driver vem no proprio token usb, basta escolher a sua plataforma e instalar ele (não 
colocar um passo a passo disso aqui, pois varia de token para token).

Depois de instalados os drivers pode ser necessário baixar a cadeia de validação deles e instalar no browser, esse passo também depende do token e da entidade
certificadora, então não há muito como dar um passo-a-passo, em geral tem-se estas informações quando você retira o seu token.

Agora o pulo do gato, a informação que eu realmente dei uma penada pra fazer funcionar. Acontece que mesmo instalando o driver do token nem tudo que é necessário é instalado no mac, é necessário também instalar um serviço para que o token seja reconhecido 
e carregado como um certificado digital, para isto basta clicar neste link: [SmartCard Services](http://smartcardservices.macosforge.org/trac/wiki/installers)

Para o e-cpf funcionar corretamente no safari e no firefox é necessário instalar o dispositivo de segurança e isso é feito apenas no firefox.

Para isso vá em 

**Preferences > Avançado > Certificados > Dispositivos de Segurança**

Depois vá em 

**Carregar**

Então escolha o nome do módulo que você quer dar e vá em **Procurar** para apontar o arquivo, este deve ser um arquivo com o nome *libwdpkcs.dylib* depois disso
basta clicar em **Ok** e pronto! O certificado digital já estará funcionando tanto no firefox quanto no chrome =)
