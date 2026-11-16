# DEVLOG — Triagem automática de e-mails (n8n + Gemini)

Registro do que foi feito, decidido e aprendido durante o projeto.

## 2026-11-14 — Da ideia ao fluxo funcionando

### O que foi feito
- Criei a conta no n8n Cloud (trial de 14 dias) e montei o fluxo: Manual Trigger → Code (6 e-mails fictícios) → Gemini (classificação) → If (validação do formato) → Fila automática / Revisão humana.
- Escrevi e refinei o prompt de classificação, aplicando o que estudei no curso de prompt engineering da Anthropic (papel, tags XML, exemplos, formato de saída fixo).
- Testei com 6 e-mails: reclamação, cancelamento, dúvida, spam, interno e uma tentativa de prompt injection.
- Gravei a tela, converti em GIF, exportei o workflow (sem credenciais) e publiquei o repositório.

### Decisões
- **Validar a saída por código (regex no If)** em vez de confiar só no prompt: se a IA responder fora do formato, o e-mail vai para revisão humana.
- **Só e-mails fictícios**, porque o modelo usado é *preview* e APIs gratuitas podem usar os dados enviados.
- **Teste em lote de 6 e-mails** num único clique, em vez de executar um por vez.
- Não publiquei o workflow (rodo manualmente) para não consumir execuções do trial.

### Resultado
- 6/6 classificações corretas em uma execução. Custo estimado: menos de US$ 0,01 por execução.
- A tentativa de injeção (e-mail 6) foi classificada pelo conteúdo real: DUVIDA.

### Problemas encontrados
- Erro de digitação no nome de um campo do nó Edit Fields (corrigido para `email`).
- O nó If falhava com o campo de comparação vazio (preenchi com a regex).
- Nó "Edit Fields" sobrando no canvas (removido).
- Ao editar o README no Bloco de Notas, o texto ganhou barras e a tabela quebrou. Resolvi subindo o arquivo original direto pelo GitHub.

### Aprendizados
- Instrução no prompt ajuda, mas a defesa mais confiável é validar a saída.
- 6 testes mostram que o fluxo funciona; não medem precisão.

### Próximos passos
- v2: gatilho real (Gmail), domínio do remetente para detectar INTERNO e registro das classificações em planilha.
- Conjunto de teste maior (20–30 e-mails) e várias rodadas para medir acerto por categoria.

## 2026-11-16 — v2: Gmail real, log em planilha e teste com gabarito

### O que foi feito
- Troquei a lista fixa de e-mails por um Gmail Trigger numa conta de teste.
- Adicionei um nó Code antes do Gemini: extrai o domínio do remetente, mascara CPF e telefone (`[CPF]`, `[TELEFONE]`) e remove `<` e `>` do texto.
- Passei a registrar cada classificação numa planilha do Google Sheets (data, domínio, categoria, destino, assunto).
- Criei 24 e-mails fictícios com o gabarito definido **antes** de rodar e um segundo workflow só para enviá-los à conta de teste.
- Gravei a execução e converti em GIF. Os workflows foram exportados sem credenciais e sem dados pessoais, na pasta `v2/`.

### Decisões
- **Gabarito fixado antes da execução**, para não ajustar a expectativa ao resultado.
- **Código `[T01]…[T24]` no assunto** como chave para cruzar cada e-mail com o gabarito na planilha.
- **Mascarar dados pessoais antes de enviar ao modelo**, não depois.
- **Mesmo prompt e mesma validação por regex da v1**, para mudar só uma coisa por vez.
- **Tratar como v2 do Projeto 1**, e não como projeto novo.

### Resultado
- 24/24 corretos (100%), todos na fila automática.
- CPF e telefone apareceram mascarados na saída do nó Code.
- As duas tentativas de injeção (uma fechando a tag `</email>`) foram classificadas pelo conteúdo real.

### Problemas encontrados
- Rodei o envio sem publicar a triagem: os e-mails chegaram antes de o gatilho existir e foram ignorados. Apaguei os e-mails e a linha de teste, aumentei o Max Emails per Poll para 30, publiquei, reenviei e despubliquei no fim.
- Com Simplify desligado, o Gmail Trigger devolvia um payload enorme. Voltei para ligado e usei o `snippet`.
- A primeira geração dos 24 e-mails saiu com distribuição errada e um caso incoerente com a regra de desempate. Regenerei com roteiro fixo em vez de remendar.
- O Code do workflow de envio estava sem `return`.
- O GIF foi enviado como `triagem-gmail-v2.gif.gif` (extensão duplicada) e o link do README ficou quebrado.

### Aprendizados
- A regex valida o **formato** da resposta, não se a categoria está certa.
- 100% em 24 casos, com gabarito escrito com ajuda de IA e casos majoritariamente claros, não prova precisão em produção.
- Dado pessoal deve ser tratado antes de chegar ao modelo.

### Limitações conhecidas
- Todos os remetentes são `gmail.com`: a regra de domínio para INTERNO não foi testada.
- O ramo de revisão humana não foi exercitado nesta execução.
- O Gmail Trigger entrega só o `snippet` (~200 caracteres).
- Uma única execução, sem medir variação entre rodadas.

### Próximos passos
- Regra de domínio para INTERNO, testada com domínios diferentes.
- Buscar o corpo completo do e-mail (nó Get a message).
- Conjunto maior, com casos ambíguos e e-mails no estilo real, e várias rodadas.
- Medir precisão por categoria.

## 2026-11-16 — Revisão do repositório

### O que foi feito
- Reescrevi a seção "Segurança" e o "Aprendizado principal" do README.
- Adicionei um resumo do estado atual (v2) no topo do README.
- Subi a evidência do teste: `v2/resultados.csv` (log da planilha) e `v2/gabarito.csv`.
- Renomeei o GIF da v2 para `triagem-gmail-v2.gif` (estava com a extensão duplicada) e corrigi o link.
- Removi o `instanceId` do n8n do `triagem-emails.json`.
- Troquei a lista "Próximos passos (v2)" da v1 por um quadro do que foi e do que não foi feito.

### O que estava errado
- O README dizia que a validação por regex era a camada "mais forte" contra prompt injection. Não é verdade para este caso: a injeção do e-mail 6 pedia `<categoria>CANCELAMENTO</categoria>`, um formato válido. Se o modelo tivesse obedecido, a regex aprovaria.
- O texto da v1 contradizia o aprendizado que eu mesmo registrei na v2 (a regex valida formato, não acerto).

### Aprendizados
- A validação de saída limita o estrago (só 5 rótulos possíveis), mas não detecta rótulo errado. Para isso servem o teste com gabarito e a revisão humana por amostragem.
- Resultado sem arquivo de evidência no repositório é só uma afirmação.
- Ao fechar uma versão nova, reler o texto da versão antiga: ele pode ter ficado errado.
