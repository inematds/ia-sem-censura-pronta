# IA sem censura pronta

**Validação atual:**14 testes de software passaram. A execução com modelo real nesta produção aguarda autorização de carga de modelo e API local; veja [o estado da validação](verificacao/STATUS.md).

## O que entrega

Este kit põe um modelo abliterated para conversar com você no próprio computador: uma versão publicada com menos restrições de resposta. O foco é executar esse modelo de verdade, não apenas explicar o assunto. O padrão é Huihui Qwen3.8-abliterated 27B. Ele também roda localmente neste kit; menos recusas e privacidade continuam sendo características diferentes.

**Você termina conversando com um modelo real.** O kit traz o lançador, o perfil do modelo, a preparação dos pesos, conversa com contexto, perguntas por arquivo, um modo Linux sem rede e uma verificação executável. Não precisa de chave, assinatura de modelo ou bibliotecas Python extras.

## Requisitos

- Python 3.10 ou superior: [instalador oficial](https://www.python.org/downloads/).
- [Ollama instalado](https://ollama.com/download), no Linux, macOS ou Windows.
- Para o modelo padrão: aproximadamente **18 GB de download**, **32 GB de RAM recomendados**, espaço livre para os pesos e recursos para o sistema. Contexto padrão2048; o consumo real depende do runtime. GPU é opcional: CPU funciona, com desempenho diferente.
- No Linux, para o modo sem rede: `bubblewrap` e `iproute2`. O modo exige namespaces de usuário permitidos pelo sistema.

Verifique antes de começar:

```bash
python3 --version
ollama --version
```

No Windows, use `py` no lugar de `python3` se necessário. Depois de instalar o Python ou Ollama, abra um terminal novo. O kit não instala pacotes com privilégios nem muda o Ollama usado por outros projetos.

## Instalar e conversar

```bash
git clone https://github.com/inematds/ia-sem-censura-pronta.git
cd ia-sem-censura-pronta
python3 ia.py diagnostico
python3 ia.py preparar
python3 ia.py verificar
python3 ia.py conversar
```

`preparar` usa internet para baixar ou conferir os pesos do modelo. Depois, `verificar` e `conversar` usam o modelo local já baixado, sem baixar automaticamente outro modelo. Os pesos são armazenados pelo próprio Ollama, não dentro do repositório.

Na conversa, digite `/limpar` para reiniciar o contexto e `/sair` para encerrar. O kit conserva apenas algumas mensagens recentes em memória e não salva histórico. Ao sair, descarrega o modelo e encerra somente sua instância e seus processos filhos. O terminal ou o sistema operacional ainda podem registrar o que aparece na tela.

## Linux: conversa com a rede bloqueada para o processo

Depois de preparar os pesos, instale os dois recursos do sistema, se faltarem:

```bash
sudo apt install bubblewrap iproute2
bash sem-rede.sh verificar
bash sem-rede.sh conversar
```

O lançador cria uma rede separada com apenas a interface local `lo`. O modelo e o cliente continuam se comunicando dentro dela; não há interface para sair à internet. O computador continua conectado normalmente para os outros programas. Esse modo isola a rede, não os arquivos do usuário; não é uma máquina virtual.

Em macOS/Windows, a conversa local funciona pelo `ia.py`; o lançador sem rede é específico de Linux. Para uma conferência manual nesses sistemas, prepare primeiro, desconecte Wi-Fi/cabo e execute `verificar`. Não declaramos esse teste realizado em sistemas que não usamos.

## Pergunta única e arquivo

```bash
python3 ia.py perguntar "Escreva três ideias para organizar uma reunião curta."
python3 ia.py perguntar --arquivo exemplos/primeiro-pedido.txt
```

Para texto sensível, prefira `--arquivo` ou entrada padrão: argumentos digitados na linha de comando podem ficar no histórico do shell. As respostas são texto; o kit não executa comandos sugeridos pela IA.

## Como saber que funcionou

```bash
python3 ia.py verificar
```

A verificação envia uma pergunta real ao modelo (`17 + 25`) e espera `42`. Salva o resultado em `.estado/verificacao.json`, com o digest dos pesos, tokens gerados e interfaces da rede. Código de saída 0 significa que esse caso passou; 1 indica resposta diferente; 2 indica erro de preparação/execução. Uma resposta correta não comprova competência geral nem ausência de recusas.

Leia também [verificacao/STATUS.md](verificacao/STATUS.md): ele distingue testes de software de inferência realmente executada. O gabarito de um segundo exercício está em `exemplos/documento-ficticio.txt`.

## O que cada escolha significa

**Privacidade:** quem recebe os dados e onde a tarefa roda. Neste kit, o pedido vai somente à instância própria do Ollama em `127.0.0.1`. Nuvem e ferramentas externas ficam desativadas. O modo Linux sem rede acrescenta uma restrição do sistema operacional.

**Sem censura:** modelo publicado pelo mantenedor com menos recusas. Este termo não promete aceitar todo pedido, acertar toda resposta ou substituir revisão humana. O kit não adiciona filtros de temas e não altera automaticamente o modelo.

## Modelo, versão e memória

O perfil usa **`huihui_ai/Qwen3.8-abliterated:27b`**, conferido em 15/11/2026. `perfil.json` registra o digest esperado dos pesos para impedir uma troca silenciosa do modelo. Se o mantenedor mudar a tag e o digest deixar de bater, a execução para: confira a nova versão antes de atualizar `digest_esperado`. Não remova a conferência só para passar.

O perfil limita o contexto a 2048 e a resposta a 512 tokens. A conversa tem janela curta; documentos longos precisam ser divididos. O tempo da primeira resposta inclui carregar os pesos. Se houver falta de memória, use uma máquina com capacidade adequada ou escolha conscientemente outro modelo local, ajustando nome e digest. Não há fallback para nuvem.

## Comportamentos importantes

- Instância própria, porta exclusiva e nuvem desativada. Porta ocupada gera erro; nunca conectamos silenciosamente a outro serviço.
- Proxies herdados não recebem os pedidos e redirecionamentos HTTP são recusados.
- Nome de modelo remoto, pesos ausentes, digest diferente e resposta vazia interrompem a execução.
- Nenhum conector, busca na web, agente que executa ferramentas ou publicação automática.
- Privacidade deste fluxo não protege contra malware, administrador do computador, captura de tela, backups ou runtime comprometido.

## Testes de software

```bash
python3 -m unittest discover -s tests -v
```

Esses testes usam um runtime simulado e não contam como teste de modelo. O teste com IA real é `verificar`.

## Documentação das ferramentas

[Ollama: execução local e nuvem](https://docs.ollama.com/faq) · [API local de conversa](https://docs.ollama.com/api/chat) · [Modelo padrão](https://ollama.com/huihui_ai/Qwen3.8-abliterated:27b).

Código do kit: MIT. Runtime e pesos seguem suas próprias licenças; os pesos não são redistribuídos aqui.
