# clavis
Frameworks: Logstash, Beats e Kibana

# 🦟 Projeto Clavis - Pipeline de Dados de Dengue

Pipeline completo para extração, processamento e análise de dados de dengue do SUS utilizando Elastic Stack.

## 📋 Visão Geral

Este projeto implementa um pipeline de dados para:
- **Extração**: Coleta dados da API de arboviroses do SUS
- **Processamento**: Limpeza e transformação dos dados via Logstash
- **Armazenamento**: Indexação no Elasticsearch
- **Visualização**: Análise via Kibana

## 🏗️ Arquitetura

```
Python Extractor → Filebeat → Logstash → Elasticsearch → Kibana
```

## 📁 Estrutura de Arquivos

```
clavis_project/
├── extract_ndjson.py          # Extrator de dados da API SUS
├── docker-compose.yml         # Orquestração de containers
├── filebeat.yml              # Configuração do Filebeat
├── logstash.conf             # Pipeline de processamento
├── dengue-template.json      # Template de mapeamento Elasticsearch
├── datafiles/                # Diretório de dados NDJSON
│   ├── dengue_UF_ANO_MES.ndjson
│   └── dengue_UF_ANO_consolidated.ndjson
└── logstash_error_logs/      # Logs de erro do Logstash
```

## 🚀 Execução Rápida

### 1. Pré-requisitos

- Docker e Docker Compose
- Python 3.8+
- Acesso à internet (para API do SUS)

### 2. Extração de Dados

```bash
# Executar o extrator de dados
python extract_ndjson.py
```

**Opções de extração:**
- **Dados consolidados por ano** (recomendado): Gera um arquivo por estado/ano
- **Dados mensais**: Gera arquivos separados por mês

### 3. Executar o Pipeline ELK

```bash
# Subir toda a stack
docker-compose up -d

# Verificar status dos serviços
docker-compose ps

# Ver logs em tempo real
docker-compose logs -f filebeat
```

### 4. Verificar Dados no Elasticsearch

```powershell
# Ver índices criados
curl.exe -X GET "http://localhost:9200/_cat/indices?v"

# Verificar documentos
curl.exe -X GET "http://localhost:9200/dengue-data-*/_search?size=1&pretty"

# Contar documentos
curl.exe -X GET "http://localhost:9200/dengue-data-*/_count?pretty"
```

### 5. Acessar Kibana

Abrir no navegador: http://localhost:5601

## ⚙️ Configuração

### Elasticsearch
- **Porta**: 9200
- **Índice padrão**: `dengue-data-YYYY.MM.dd`
- **Health Check**: Disponível em `http://localhost:9200`

### Kibana
- **Porta**: 5601
- **Configuração**: Conecta automaticamente ao Elasticsearch

### Logstash
- **Porta**: 5044
- **Pipeline**: Processa dados NDJSON do Filebeat
- **Template**: Aplica mapeamento customizado para dados de dengue

### Filebeat
- **Input**: Monitora arquivos NDJSON em `./datafiles/`
- **Output**: Envia para Logstash na porta 5044

## 🎯 Funcionalidades do Pipeline

### Processamento de Dados
- ✅ Parse automático de JSON
- ✅ Limpeza de campos desnecessários
- ✅ Conversão de tipos (datas, números)
- ✅ Validação de dados
- ✅ Tratamento de erros

### Campos Principais Processados
- **Identificação**: `id_agravo`, `id_municip`, `sg_uf_not`
- **Datas**: `dt_notific`, `dt_sin_pri`, `dt_encerra`
- **Demográficos**: `cs_sexo`, `nu_idade_n`, `cs_raca`
- **Clínicos**: `febre`, `cefaleia`, `mialgia`, `exantema`
- **Evolução**: `classi_fin`, `evolucao`, `hospitaliz`

## 🔧 Troubleshooting

### Problemas Comuns

1. **Elasticsearch não sobe**
   ```bash
   docker-compose logs elasticsearch
   # Verificar se há conflito de porta 9200
   ```

2. **Filebeat não envia dados**
   ```bash
   docker-compose logs filebeat
   # Verificar permissões dos arquivos .ndjson
   ```

3. **Logstash com erro de parse**
   ```bash
   docker-compose logs logstash
   # Verificar estrutura dos arquivos NDJSON
   ```

### Comandos Úteis

```bash
# Reiniciar serviços específicos
docker-compose restart filebeat logstash

# Parar todos os serviços
docker-compose down

# Ver espaço em disco dos containers
docker system df

# Limpar dados não utilizados
docker system prune
```

### Verificação de Saúde

```bash
# Health check do Elasticsearch
curl.exe -X GET "http://localhost:9200/_cluster/health?pretty"

# Stats do Filebeat
docker-compose exec filebeat filebeat test output

# Status do pipeline Logstash
curl.exe -X GET "http://localhost:9600/?pretty"
```

## 📊 Análise no Kibana

Após a ingestão dos dados:

1. **Criar Index Pattern**:
   - Acessar Stack Management → Index Patterns
   - Criar pattern: `dengue-data-*`

2. **Explorar dados**:
   - Usar Discover para visualizar documentos
   - Criar visualizações em Dashboard
   - Utilizar Lens para análises rápidas

3. **Exemplos de análise**:
   - Casos por UF e período
   - Distribuição por sexo e faixa etária
   - Evolução temporal dos casos
   - Correlação com sintomas

## 🗂️ Estrutura de Dados

### Arquivos Gerados
- **Formato**: NDJSON (Newline Delimited JSON)
- **Codificação**: UTF-8
- **Localização**: `./datafiles/`
- **Padrão de nome**: `dengue_UF_ANO_consolidated.ndjson`

### Metadados do Pipeline
- `processed_at`: Timestamp de processamento
- `pipeline_version`: Versão do pipeline
- `data_source`: Fonte dos dados (SUS_Dengue)

## 📝 Notas de Desenvolvimento

### Para Modificar o Pipeline

1. **Alterar processamento**: Editar `logstash.conf`
2. **Modificar mapeamento**: Atualizar `dengue-template.json`
3. **Ajustar extração**: Modificar `extract_ndjson.py`
4. **Reconfigurar coleta**: Ajustar `filebeat.yml`

### Para Adicionar Novos Campos

1. Atualizar o template do Elasticsearch
2. Modificar o pipeline do Logstash
3. Reiniciar a stack

## 🔒 Considerações de Segurança

- Stack executada localmente
- Sem autenticação habilitada (ambiente de desenvolvimento)
- Dados epidemiológicos públicos
- Recomendado usar x-pack security para produção

## 📞 Suporte

Em caso de problemas:
1. Verificar logs com `docker-compose logs [serviço]`
2. Validar estrutura dos arquivos NDJSON
3. Testar conectividade entre serviços
4. Verificar recursos do sistema (memória, disco)

---

**📊 Status do Pipeline**: ✅ Operacional  
**🔄 Última Atualização**: Novembro 2024  
**🐛 Issues Conhecidos**: Nenhum
