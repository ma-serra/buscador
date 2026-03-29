# Buscador de Carros

API em Python/Flask que agrega anúncios de veículos de múltiplos marketplaces brasileiros em um único endpoint. O sistema realiza web scraping e retorna os dados estruturados em JSON.

## 📦 O que o aplicativo faz

O **Buscador** acessa sites de venda de carros (como OLX, iCarros, WebMotors, etc.), extrai os anúncios e devolve uma lista padronizada de veículos com os campos:

| Campo            | Descrição                              |
| ---------------- | -------------------------------------- |
| `marca`          | Marca do carro (ex: `FIAT`)            |
| `modelo`         | Modelo do carro (ex: `Uno 1.0`)        |
| `ano`            | Ano de fabricação (ex: `2023`)         |
| `preco`          | Preço em reais (ex: `35000.00`)        |
| `quilometragem`  | Quilometragem (ex: `50000`)            |
| `combustivel`    | Tipo de combustível (ex: `Flex`)       |
| `cambio`         | Câmbio (ex: `Manual`, `Automático`)    |
| `localizacao`    | Cidade/estado (ex: `São Paulo`)        |
| `descricao`      | Descrição completa do anúncio          |
| `imagem`         | URL da imagem principal                |
| `link`           | Link direto para o anúncio             |
| `site_icon`      | Ícone/favicon do site de origem        |

---

## ✅ Situação atual — O que está pronto

### Sites implementados (6 de 6 planejados inicialmente)

| Site                   | Crawler        | Parser         | Observações                                        |
| ---------------------- | -------------- | -------------- | -------------------------------------------------- |
| **OLX**                | ✅ `crawlers/olx.py`      | ✅ `parsers/olx.py`      | Playwright + stealth mode                  |
| **iCarros**            | ✅ `crawlers/icarros.py`  | ✅ `parsers/icarros.py`  | Converte sigla de estado para cidade       |
| **WebMotors**          | ✅ `crawlers/webmotors.py`| ✅ `parsers/webmotors.py`| Usa API JSON (sem scraping HTML)           |
| **NaPista**            | ✅ `crawlers/napista.py`  | ✅ `parsers/napista.py`  | Dados via JSON-LD na página                |
| **Facebook Marketplace**| ✅ `crawlers/facebook.py`| ✅ `parsers/facebook.py` | Fecha modal de login automaticamente       |
| **Mobiauto**           | ✅ `crawlers/mobiauto.py` | ✅ `parsers/mobiauto.py` | Navegação com rolagem de página            |

### Estrutura do projeto

```
buscador/
│
├── app.py               # API Flask principal
├── crawlers/            # Módulos de acesso aos sites (6 implementados)
│   ├── olx.py
│   ├── icarros.py
│   ├── webmotors.py
│   ├── napista.py
│   ├── facebook.py
│   └── mobiauto.py
│
├── parsers/             # Módulos de extração de dados (6 implementados)
│   ├── olx.py
│   ├── icarros.py
│   ├── webmotors.py
│   ├── napista.py
│   ├── facebook.py
│   └── mobiauto.py
│
├── requirements.txt
└── README.md
```

---

## ❌ O que ainda falta

### Sites não implementados

Os seguintes sites estavam no planejamento mas ainda não possuem crawler nem parser:

| Site             | Arquivo esperado           |
| ---------------- | -------------------------- |
| MeuCarroNovo     | `crawlers/meu_carro_novo.py` / `parsers/meu_carro_novo.py` |
| AutoAvaliar      | `crawlers/autoavaliar.py` / `parsers/autoavaliar.py`       |
| RepassaFácil     | `crawlers/repassafacil.py` / `parsers/repassafacil.py`     |
| EstaCarro        | `crawlers/estacarro.py` / `parsers/estacarro.py`           |
| Volat            | `crawlers/volat.py` / `parsers/volat.py`                   |
| VAPT             | `crawlers/vapt.py` / `parsers/vapt.py`                     |

### Outras melhorias pendentes

- Suporte a paginação (atualmente retorna apenas a primeira página)
- Testes automatizados (pytest)
- Validação e sanitização dos dados extraídos
- Logging estruturado
- Configuração para deploy em produção (Docker, WSGI)
- Módulo `utils/helpers.py` para funções auxiliares compartilhadas (limpeza de dados, formatação)

---

## 🚀 Como instalar e executar

### Pré-requisitos

- Python 3.10+
- Chromium (instalado via Playwright)

### Instalação

```bash
pip install -r requirements.txt
playwright install chromium
```

### Executar a API

```bash
python app.py
```

A API ficará disponível em `http://localhost:5000`.

---

## 📘 Documentação da Rota `/get-dom`

### 🔗 Endpoint

```
GET /get-dom
```

### 📌 Parâmetros

| Parâmetro | Obrigatório         | Descrição                                                    |
| --------- | ------------------- | ------------------------------------------------------------ |
| `site`    | ✅ Sempre            | Nome do site em minúsculo (ex: `olx`, `webmotors`)          |
| `estado`  | ✅ Maioria dos sites | Sigla do estado (ex: `sp`, `rj`). Não é usado no NaPista.   |
| `carro`   | ✅ Maioria dos sites | Modelo ou nome do carro (ex: `kwid`, `civic`). **Não usado no WebMotors** — substitua por `modelo`. |
| `marca`   | ⚠️ WebMotors e Mobiauto | Marca do carro (ex: `fiat`, `chevrolet`)                |
| `modelo`  | ⚠️ WebMotors        | Substitui `carro` no WebMotors (ex: `modelo=uno` em vez de `carro=uno`) |

---

## 🟢 Exemplos por Site

### 🔹 OLX

```
GET /get-dom?site=olx&estado=sp&carro=kwid
```

### 🔹 iCarros

```
GET /get-dom?site=icarros&estado=mg&carro=uno
```

### 🔹 Mobiauto

```
GET /get-dom?site=mobiauto&estado=sp&marca=chevrolet&carro=onix
```

### 🔹 WebMotors

```
GET /get-dom?site=webmotors&estado=rj&marca=fiat&modelo=uno
```

### 🔹 NaPista

> Não precisa do estado, apenas do modelo do carro.

```
GET /get-dom?site=napista&carro=gol
```

### 🔹 Facebook Marketplace

```
GET /get-dom?site=facebook&estado=sp&carro=civic
```

---

## 📝 Observações

- Os nomes dos sites devem estar em **minúsculo** (ex: `webmotors`, `facebook`).
- O parâmetro `marca` é obrigatório para os sites: `webmotors` e `mobiauto`.
- O `estado` deve estar no formato de sigla: `sp`, `rj`, `am` etc.
- A cidade é automaticamente derivada da sigla do estado pelos crawlers.
