# Trabalho realizado nas Semanas #2 e #3


## Identificação

- O ProxyLogon (CVE-2021-26855) é uma vulnerabilidade que permite escalar privilégios num servidor (windows) que corra o Microsoft Exchange.
- A vulnerabilidade afeta certas versões do Microsoft Exchange 2013, 2016 e 2019.
- Esta permite ao atacante dar *bypass* do mecanismo de autenticação e fazer pedidos com privilégios.

## Catalogação

- Descoberto por Orange Tsai, da equipa [DEVCORE](https://devco.re/en/) , a 10 de dezembro de 2020.
- Foi divulgada, em privado, em 5 de janeiro de 2021 à Microsoft, e divulgado ao público no dia 3 de março de 2021.
- Categorizada com 9.8 na escala de NIST, devido à sua baixa complexidade.
- Não há *bug bounty* associada publicamente.

## Exploit

- Neste CVE um usuário externo consegue dar *bypass* do mecanismo de autenticação através de um Server-Side Request Forgery (SSRF).
- O cookie `X-BEResource` é lido pelo *proxy* (IIS) que redireciona os pedidos para o backend do MS Exchange.
- A vulnerabilidade em questão é causada pelo `UriParser` de C# não fazer a validação correta do host do URI.
- Este POST é redirecionado para o serviço referido no cookie `X-BEResource`, o que dá acesso com permissões de administrador a qualquer serviço dentro do MS Exchange.

## Ataques

- Antes da vulnerabilidade ser divulgada ao público esta já estava a ser usada por grupos maliciosos.
- Como esta vulnerabilidade faz com que qualquer pedido tenha as permissões mais altas, foram descobertas outras vulnerabilidades dentro do *backend* do Microsoft Exchange.
- Associadas a esta vulnerabilidade, existem também o CVE-2021-26857 e CVE-2021-26858.
- Também existe um módulo do [Metasploit](https://packetstormsecurity.com/files/162736/Microsoft-Exchange-ProxyLogon-Collector.html) que consegue automatizar esta vulnerabilidades.
