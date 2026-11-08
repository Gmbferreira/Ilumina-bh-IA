# IluminaBH

**Sistema Inteligente de Análise da Iluminação Pública de Belo Horizonte**

Projeto do Time de Engenharia de IA | PUC Minas | Entrega final: 06/12/2026

Integrantes: Guilherme Martini Brina Ferreira e Pedro Rodrigues

Repositório: https://github.com/Gmbferreira/Ilumina-bh-IA

---

## Sobre o projeto

O IluminaBH é uma aplicação web, executada localmente, que integra dados públicos de iluminação pública de Belo Horizonte, trata e armazena esses dados e permite explorá-los por mapas, filtros, indicadores e relatórios. Como componente de IA, usa aprendizado de máquina não supervisionado (agrupamento com K-Means) para identificar **perfis semelhantes** da infraestrutura.

### O que a IA faz e o que ela não faz

A IA identifica padrões nos dados. Ela **não** detecta lâmpadas queimadas, não prevê falhas, consumo ou manutenção e não avalia a qualidade da iluminação. Um perfil é uma estrutura encontrada nos dados segundo os atributos escolhidos, nunca um diagnóstico oficial nem um julgamento de "bom" ou "ruim".

## Funcionalidades

- Importação, validação, limpeza e padronização das bases da PBH
- Armazenamento local em SQLite, com verificação de integridade (lidos = aceitos + rejeitados)
- Mapa interativo com unidades (pontos) e trechos (linhas), camadas ativáveis e consulta de atributos
- Filtros por regional, bairro, tipo de logradouro, LED, telegestão, objeto iluminado e iluminação de destaque
- Indicadores gerais, por regional e por bairro, com exportação em CSV
- Relatório de qualidade dos dados (ausentes, duplicados, IDs e geometrias inválidas)
- Pipeline de ML: engenharia de atributos, K-Means, avaliação de k (inércia, silhueta, Davies-Bouldin, estabilidade) e caracterização dos perfis
- Registro de cada execução (hash dos dados, atributos, parâmetros, semente, versões e métricas) para reprodutibilidade

## Fontes de dados

Portal de Dados Abertos da Prefeitura de Belo Horizonte (PBH):

| Base | Tipo | Uso |
|------|------|-----|
| `UNIDADE_ILUMINACAO_PUBLICA` | Pontos | Mapa, filtros, indicadores e única base do modelo de IA |
| `TRECHO_ILUM_PUBLICA` | Linhas | Mapa e contexto geográfico complementar |

São dados abertos, sem dados pessoais. As bases não informam falhas, consumo, potência, data de instalação nem estado de conservação, e podem estar incompletas; a completude é reportada no relatório de qualidade.

## Arquitetura

```
Dados PBH -> Ingestão -> Validação e limpeza -> SQLite
                                                  |
                          +-----------------------+-----------------------+
                          |                                               |
                    API (FastAPI)                                 Pipeline de ML
                  consultas e filtros                    atributos -> K-Means -> avaliação
                          |                                               |
                          +-----------------------+-----------------------+
                                                  |
                                          Aplicação web
                                  Mapa | Relatórios | Análise IA
```

## Tecnologias

| Camada | Tecnologia |
|--------|-----------|
| Linguagem | Python 3 |
| Dados e geometrias | pandas, GeoPandas, Shapely |
| Machine Learning | scikit-learn |
| Banco | SQLite (PostgreSQL/PostGIS como evolução possível) |
| Backend | FastAPI, Uvicorn |
| Frontend | HTML, CSS, JavaScript, Leaflet, Plotly |
| Testes e CI | pytest, GitHub Actions |

Tudo gratuito, sem GPU, sem nuvem e sem serviços pagos.

## Instalação e execução

> Os comandos abaixo seguem a organização planejada. Ajuste os nomes de módulos e scripts conforme a implementação final.

**Pré-requisitos:** Python 3.10+ e Git.

```bash
# 1. Clonar o repositório
git clone https://github.com/Gmbferreira/Ilumina-bh-IA.git
cd Ilumina-bh-IA

# 2. Criar o ambiente virtual e instalar as dependências
python -m venv .venv
source .venv/bin/activate      # Windows: .venv\Scripts\activate
pip install -r requirements.txt

# 3. Colocar os arquivos baixados do Portal de Dados Abertos da PBH
#    na pasta de dados brutos (ver data/README ou a seção de dados)

# 4. Importar, validar e gravar os dados
# python -m <modulo_de_carga>

# 5. Executar o pipeline de ML
# python -m <modulo_de_ml> --seed 42

# 6. Iniciar o servidor local
# uvicorn <modulo_api>:app --host 127.0.0.1 --port 8000
```

Depois, abra http://127.0.0.1:8000 no navegador. O servidor aceita conexões apenas do próprio computador.

Sem internet, o sistema continua funcionando: o mapa-base do OpenStreetMap é opcional e as camadas do projeto funcionam offline.

## Testes

```bash
pytest
```

Os testes cobrem regras de validação, indicadores e reprodutibilidade do agrupamento, e rodam automaticamente no GitHub Actions a cada commit e pull request.

## Estrutura do repositório (planejada)

```
Ilumina-bh-IA/
├── data/            # dados brutos (sem alteração) e banco SQLite
├── ingestao/        # leitura dos arquivos
├── tratamento/      # validação, limpeza e padronização
├── armazenamento/   # persistência no SQLite
├── ml/              # engenharia de atributos, K-Means e avaliação
├── api/             # FastAPI
├── web/             # interface (HTML, CSS, JS)
├── tests/           # testes automatizados
└── docs/            # documentação
```

## Roadmap

| Período | Fases |
|---------|-------|
| 08/11 a 14/11 | Carregar dados, limpar e validar, banco de dados |
| 15/11 a 21/11 | Mapa, filtros e relatórios |
| 22/11 a 28/11 | Engenharia de atributos, agrupamento e avaliação, perfis no mapa |
| 29/11 a 06/12 | Testes, documentação e Release v1.0 |

**Trabalhos futuros:** migração controlada para PostgreSQL/PostGIS, caso as medições de desempenho mostrem que o SQLite não atende consultas geográficas interativas.

## Limitações conhecidas

- Os atributos são majoritariamente categóricos, o que limita o K-Means; ele serve como linha de base interpretável.
- Os perfis não refletem qualidade nem mérito de uma região; pode haver viés de cobertura em regiões com cadastro menos completo.
- A escolha final de k é uma decisão humana e documentada, não automática.

## Princípios éticos

Transparência e explicabilidade (sem caixa-preta), privacidade (dados públicos, processamento local), supervisão humana (a IA apoia a análise, não decide) e sustentabilidade (modelo leve, em CPU). Detalhes no Manifesto Ético do Time.

## Licença

A definir pela equipe. Verifique também os termos de uso dos dados no Portal de Dados Abertos da PBH.
