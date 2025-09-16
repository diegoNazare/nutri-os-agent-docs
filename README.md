# Nutri-OS Agent Documentation

Este repositório contém toda a documentação e diretrizes para assistentes de IA (Lovable e Cursor) que trabalham no projeto Nutri-OS. É projetado para ser usado como um submodule compartilhável entre múltiplos projetos.

## 📁 Estrutura do Repositório

```
.agents/
├── README.md                    # Este arquivo - visão geral do repositório
├── PROJECT_GUIDELINES.md        # Diretrizes principais do projeto
├── guidelines/                  # Diretrizes específicas por área
│   ├── ARCHITECTURE.md         # Arquitetura e estrutura de código
│   ├── GIT_BRANCHING.md        # Workflow do Git e branching
│   ├── LOCAL_DEVELOPMENT.md    # Ambiente de desenvolvimento local
│   ├── PRODUCTION.md           # Deploy e produção
│   └── UI_GUIDELINES.md        # Padrões de UI/UX
└── troubleshooting/            # Soluções para problemas conhecidos
    └── MODAL_FREEZE_FIX.md     # Solução para congelamento de modais
```

## 🎯 Propósito

Este repositório serve como uma **fonte única de verdade** para:

- **Diretrizes de desenvolvimento** para assistentes de IA
- **Padrões de código** e arquitetura
- **Workflows** e processos estabelecidos
- **Soluções** para problemas técnicos conhecidos
- **Padrões de UI/UX** e design system

## 🚀 Como Usar

### Para Assistentes de IA (Lovable/Cursor)

1. **Consulte sempre** o arquivo relevante antes de fazer mudanças
2. **Siga as diretrizes** específicas para cada área (UI, arquitetura, etc.)
3. **Mantenha consistência** com os padrões estabelecidos
4. **Nunca faça commits** sem solicitação explícita do usuário

### Para Desenvolvedores

1. **Leia** o `PROJECT_GUIDELINES.md` para visão geral
2. **Consulte** as diretrizes específicas conforme necessário
3. **Atualize** a documentação quando padrões mudarem
4. **Reporte** inconsistências ou sugestões de melhoria

## 📋 Diretrizes Principais

### 🎨 UI/UX
- **SEMPRE** use componentes shadcn/ui
- **NUNCA** use emojis na interface
- Mantenha padrões de acessibilidade
- Siga o design system estabelecido

### 💻 Desenvolvimento
- Use Supabase local para desenvolvimento
- Siga padrões de TypeScript rigorosamente
- Implemente tratamento de erros adequado
- Mantenha componentes focados e reutilizáveis

### 🌿 Git & Versionamento
- **TODAS** as operações Git em inglês
- **NUNCA** faça commits sem solicitação explícita
- Siga estratégia de branching estabelecida
- Use mensagens de commit descritivas

### 🏗️ Arquitetura
- React + TypeScript + Supabase
- Componentes funcionais com hooks
- Estado global com Zustand quando necessário
- Custom hooks para lógica de negócio

## 🔧 Stack Tecnológica

### Frontend
- **React 18** com TypeScript
- **Vite** para build
- **Tailwind CSS** + **shadcn/ui** para UI
- **React Router** para navegação

### Backend
- **Supabase** (PostgreSQL + Edge Functions)
- **Row Level Security** para proteção de dados
- **Real-time subscriptions** para atualizações ao vivo

### Ferramentas
- **ESLint** para linting
- **Prettier** para formatação
- **TypeScript** para type safety

## 🐛 Troubleshooting

### Problemas Conhecidos
- **Modal Freeze**: Solução documentada em `troubleshooting/MODAL_FREEZE_FIX.md`
- **Focus Scope Conflicts**: Use `onSelect` com `setTimeout` para dropdowns que abrem modais

### Como Reportar Problemas
1. Verifique se já existe solução na pasta `troubleshooting/`
2. Documente o problema com detalhes específicos
3. Inclua passos para reproduzir
4. Sugira soluções quando possível

## 🔄 Manutenção

### Atualizando Diretrizes
- Diretrizes são documentos vivos
- Atualize quando padrões mudarem
- Documente novas convenções
- Mantenha todos os arquivos sincronizados

### Versionamento
- Rastreie mudanças nos arquivos de diretrizes
- Use mensagens de commit descritivas
- Revise mudanças antes de fazer merge
- Mantenha diretrizes sincronizadas com o código

## 📚 Referência Rápida

| Área | Arquivo | Descrição |
|------|---------|-----------|
| **Visão Geral** | `PROJECT_GUIDELINES.md` | Diretrizes principais e referência rápida |
| **UI/UX** | `guidelines/UI_GUIDELINES.md` | Padrões de interface e design |
| **Arquitetura** | `guidelines/ARCHITECTURE.md` | Estrutura de código e padrões |
| **Desenvolvimento** | `guidelines/LOCAL_DEVELOPMENT.md` | Setup e ambiente local |
| **Git** | `guidelines/GIT_BRANCHING.md` | Workflow e branching |
| **Produção** | `guidelines/PRODUCTION.md` | Deploy e ambiente de produção |
| **Problemas** | `troubleshooting/` | Soluções para problemas conhecidos |

## 🤝 Contribuindo

### Para Assistentes de IA
- Sempre consulte as diretrizes antes de fazer mudanças
- Mantenha consistência com padrões estabelecidos
- Documente novas soluções na pasta `troubleshooting/`
- Nunca faça commits sem solicitação explícita

### Para Desenvolvedores
- Mantenha a documentação atualizada
- Reporte inconsistências ou melhorias
- Siga os padrões estabelecidos
- Contribua com soluções para problemas técnicos

## 📞 Suporte

### Dúvidas
1. Consulte o arquivo de diretrizes relevante
2. Procure exemplos no código existente
3. Peça esclarecimentos quando necessário
4. Documente novos padrões conforme surgem

### Problemas
- Reporte inconsistências nas diretrizes
- Sugira melhorias nos processos
- Atualize documentação conforme necessário
- Mantenha os padrões de qualidade

---

**Última atualização**: $(date)  
**Versão**: 1.0.0  
**Mantenedor**: Equipe Nutri-OS
