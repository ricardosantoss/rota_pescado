# Rota do Pescado · Acesso à inspeção de pescado

Mapa interativo que mostra quanto tempo de estrada separa cada município de um estabelecimento de pescado com inspeção sanitária, federal (SIF) ou estadual (SIE), e quanto da produção de peixe criado fica a até um limite de horas.

Site estático, sem build: HTML, CSS e JavaScript puro, com o Leaflet carregado por CDN. Todos os cálculos rodam no navegador.

## Publicar na Vercel

1. Crie um repositório no GitHub e envie esta pasta:
   ```bash
   git init
   git add .
   git commit -m "Primeira versão do mapa de acesso à inspeção"
   git branch -M main
   git remote add origin https://github.com/<seu-usuario>/rota-pescado-inspecao.git
   git push -u origin main
   ```
2. Na Vercel: **Add New → Project → Import** o repositório.
3. Em *Framework Preset*, escolha **Other**. Deixe *Build Command* e *Output Directory* em branco e clique em **Deploy**.

A cada `git push`, a Vercel publica a nova versão sozinha.

Para testar no computador antes de publicar:
```bash
python -m http.server 8000
# abra http://localhost:8000
```
Abrir o `index.html` direto do disco não funciona, porque o navegador bloqueia a leitura dos arquivos JSON fora de um servidor.

## O que o site faz

- **Camadas:** só federal (SIF), só estadual (SIE) ou as duas.
- **Limite de viagem e fator de caminhão:** recalculam a cobertura do estado na hora.
- **Linha do tempo de 2015 a 2025:** produção de cada ano da PPM. Cada estabelecimento entra a partir do ano de registro (SIE pelo número, ex.: 1490/15 = 2015; SIF pela data de reserva no SIGSIF). O botão ▶ anima os anos.
- **Ficha do município:** produção e série histórica, participação da tilápia, reservatório e cessões em águas da União, SIF e SIE mais próximos, os 6 estabelecimentos mais perto e o botão "ver rota", que desenha o caminho pela estrada (OSRM).
- **Simulador "e se":** clique em municípios para pôr unidades hipotéticas, ou peça a melhor próxima unidade (heurística gulosa da máxima cobertura).
- **Tabela de municípios produtores:** com busca, ordenação e download em CSV.
- **Link compartilhável:** o endereço guarda camada, limite, ano, município e simulação (`#uf=GO&camada=SIE&T=2&ano=2025&mun=5214606`).

## Estrutura

```
index.html, css/, js/app.js     o site
data/estados.json               estados no seletor (ativo: true/false)
data/GO/                        dados que o site lê
  municipios.json               produção 2015–2025, região, reservatório, tilápia, pescadores
  estabelecimentos.json         SIF e SIE (nome, número, município, ano, avisos)
  matriz.json                   tempos de carro (min) e distâncias (km) entre todos os nós
  municipios.geojson            malha municipal simplificada
  meta.json                     fontes, datas, nós de fora do estado
insumos/                        arquivos de origem (não vão para o site)
scripts/build_data.py           gera data/<UF>/ a partir de insumos/
```

## Atualizar os dados ou acrescentar um estado

1. Ponha em `insumos/<UF>/` os mesmos arquivos de Goiás:
   - a matriz de tempos e a base municipal, que saem do notebook do artigo;
   - a série da PPM;
   - a lista de SIE do órgão estadual;
   - a malha municipal.
2. Rode:
   ```bash
   pip install pandas geopandas shapely
   python scripts/build_data.py AP
   ```
3. Em `data/estados.json`, troque `"ativo": false` para `true`.
4. Faça `git push`.

No Amapá, a matriz deve incluir o tempo por rio onde não há estrada. O notebook de rotas já tem essa opção.

## Fontes

| Dado | Fonte |
|---|---|
| Produção de peixe criado | IBGE, Pesquisa da Pecuária Municipal, tabela 3940 |
| SIF de pescado | MAPA, SIGSIF (dados abertos), consulta em 24/11/2026 |
| SIE de pescado | Agrodefesa, consulta pública (LAI), em 23/11/2026 |
| Tempos de viagem | OSRM sobre o OpenStreetMap |
| Malha municipal | geodata-br, a partir do IBGE |
| Coordenadas das sedes | kelvins/municipios-brasileiros |
| Cessões em águas da União | MPA, tabulação da equipe do projeto |

Telefones, e-mails e CPF/CNPJ dos estabelecimentos não são publicados no site.

## Cuidados

- Cada município é representado pela sede; o tempo real até os tanques pode ser maior.
- Os cadastros listam só quem está ativo hoje, então fechamentos passados não aparecem na linha do tempo.
- Alguns estabelecimentos estão em recuperação judicial, e a categoria "pescado" do SIF inclui um ranário. Os dois casos aparecem marcados.
- O simulador indica onde a cobertura cresce mais. Isso não é estudo de viabilidade.
