---
title: Acessando a API
keywords: pedidos, pagamentos, split de pagamentos, autenticacao
last_updated: July 3, 2016
tags: [pedidos, pagamentos, split, autenticacao]
summary: "Veja como acessar suas chaves de autenticação para usar a API."
sidebar: mydoc_sidebar
permalink: acessando-a-api.html
folder: pagamentos
---

## Introdução
Sua conta possui 3 chaves:

uma chave de acesso **(access token)**;
uma chave de atualização **(refresh token)**;
e uma chave de cliente **(client token)**.

Você utiliza a chave de acesso para acessar a API, realizar pagamentos, atualizar seus dados, cadastrar webhooks etc. Essa chave tem a validade de 1 semana. Após expirar você utiliza a chave de atualização **(refresh token)** para gerar uma nova chave de acesso no endpoint de autenticação.

A chave de cliente **(client token)** você irá utilizar no endpoint de tokenizar cartão de crédito no momento do pagamento para gerar um token do cartão de crédito. Para saber mais sobre pagamentos com cartão de crédito consulte nosso guia Cobrando no Crédito.

## Gerando uma nova chave de atualização
A chave de atualização **(refresh token)** não expira, caso você perca essa chave ou ela esteja comprometida, você pode gerar uma nova chave na dashboard da sua conta. Para isso, entre na plataforma e clique no seu perfil no canto superior direito e depois em **Perfil e Preferências**:

{% include image.html file="perfil-preferencias.png" caption="Acessando preferências da conta." %}

Em seguida no menu lateral esquerdo clique em **Desenvolvedor > Gerar nova chave**:

{% include image.html file="desenvolvedor.png" caption="Gerando uma nova chave de atualização." %}

Copie a nova chave gerada:

{% include image.html file="refresh-token.png" caption="Chave de atualização." %}

## Atualizando sua chave de acesso
Para atualizar ou gerar sua chave de acesso, faça uma requisição POST no endpoint de autenticação.

[Referência API: Autenticação](https://docs.gen.com.br/#90042646-b312-4fe8-a5b1-1d53122ef5c9)
```bash
curl -X POST 'https://api-sandbox.genpag.com.br/api/:sellerId/tokens' --data '{}' -H 'Authorization: Bearer (refresh_token)'
# Resposta:
{"data": "SFMyNTY..."}
```


O parâmetro :sellerId você encontra acessando a plataforma web, e indo no seu perfil e clicando em **Perfil e preferências > Desenvolvedor > Chaves de Acesso > Meu ID**

Envie um cabeçalho Authorization com o conteúdo **Bearer (refresh_token)** onde refresh_token é a sua chave de atualização conseguida no passo anterior. Utiliza essa chave para gerar pedidos e aprovar pagamentos. Quando receber uma resposta com status 401 (Não Autorizado) realize a atualização da chave de acesso e tente a requisição novamente.

## Chave de cliente
A sua chave de cliente pode ser encontrada na plataforma web clicando no seu perfil **Perfil e preferências > Desenvolvedor > Chave do cliente**.

Utilize essa chave para autenticar no endpoint de tokenizar cartão para gerar um token dos dados do cartão de crédito do seu cliente.

Você deve realizar essa chamada a partir da sua aplicação cliente diretamente do browser, app mobile ou aplicação desktop. Você não deve trafegar os dados do cartão de crédito através de seus servidores. Para manter o escopo de segurança, a requisição deve partir diretamente do dispositivo do usuário. Após a tokenização, você pode utilizar o token gerado normalmente nas APIs de pagamento.

## Próximos Passos
Pronto, agora que você já tem as chaves da sua conta em mãos veja os próximos passos:

* Veja como gerar pedidos através da nossa API utilizando as credenciais da sua conta. [Ver Tutorial]({% link pages/pagamentos/gerando_pedidos.md %});
* Aprenda a gerar boletos para pedidos geradas. Ver Tutorial;
* Aprenda a gerar PIX  de cobrança. Ver Tutorial;
* Veja como cobrar um pedido no cartão de crédito. Ver Tutorial;
* Cadastre webhooks para receber notificações de eventos da nossa API no seu sistema. Ver tutorial.

Se você pretende construir um marketplace e gerenciar pedidos entre múltiplos estabelecimentos com split de pagamentos, não deixe de conferir nosso guia Cadastrando um Aplicativo.
