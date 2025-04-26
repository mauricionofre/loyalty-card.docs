---
layout: default
title: Documentação do Loyalty Card
nav_order: 1
permalink: /
---

# Documentação do Loyalty Card

Bem-vindo à documentação oficial do sistema Loyalty Card. Esta documentação contém informações sobre a arquitetura, APIs, modelos de domínio e procedimentos operacionais do sistema.

## Visão Geral do Projeto

O Loyalty Card é uma plataforma SaaS multi-tenant para gestão de programas de fidelidade para empresas de diversos segmentos. A plataforma permite:

- Gerenciar programas de fidelidade personalizados
- Acompanhar pontos e resgates de clientes
- Configurar regras de negócio específicas para cada segmento
- Automatizar comunicações com clientes

## Navegação da Documentação

A documentação está organizada nas seguintes seções:

<div class="docs-grid">
  <a href="/docs/architecture/" class="docs-card">
    <h3>Arquitetura</h3>
    <p>Arquitetura de referência e decisões arquiteturais do sistema</p>
  </a>
  <a href="/docs/diagrams/" class="docs-card">
    <h3>Diagramas</h3>
    <p>Diagramas de sequência, estado e arquitetura do sistema</p>
  </a>
  <a href="/docs/api/" class="docs-card">
    <h3>API</h3>
    <p>Documentação da API REST e endpoints disponíveis</p>
  </a>
  <a href="/docs/domain/" class="docs-card">
    <h3>Modelo de Domínio</h3>
    <p>Entidades, agregados e regras de negócio</p>
  </a>
  <a href="/docs/operations/" class="docs-card">
    <h3>Operações</h3>
    <p>Guias de operação, backup e disaster recovery</p>
  </a>
  <a href="/docs/requirements/" class="docs-card">
    <h3>Requisitos</h3>
    <p>Requisitos funcionais e não-funcionais</p>
  </a>
</div>

## Primeiros Passos

Para começar a explorar o sistema, recomendamos seguir esta ordem:

1. Entenda a [Arquitetura Geral](/docs/architecture/)
2. Explore o [Modelo de Domínio](/docs/domain/)
3. Consulte a [Documentação da API](/docs/api/)
4. Analise os [Diagramas do Sistema](/docs/diagrams/)

## Status Atual

Este projeto está atualmente em fase de **MVP (Produto Mínimo Viável)**, com foco em implementar as funcionalidades essenciais. Partes da documentação podem refletir funcionalidades planejadas para versões futuras.

<div class="callout info">
  <p class="callout-title">Nota sobre o MVP</p>
  <p>As decisões arquiteturais para o MVP priorizam velocidade de implementação e simplicidade operacional. A documentação indica claramente quais funcionalidades estão disponíveis no MVP e quais estão planejadas para o futuro.</p>
</div>

<style>
.docs-grid {
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(280px, 1fr));
  gap: 1.5rem;
  margin: 2rem 0;
}

.docs-card {
  background: rgba(115, 115, 240, 0.05);
  border: 1px solid #2d2d4a;
  padding: 1.2rem;
  border-radius: 8px;
  transition: all 0.3s ease;
  color: inherit !important;
  text-decoration: none !important;
  display: flex;
  flex-direction: column;
}

.docs-card:hover {
  transform: translateY(-3px);
  box-shadow: 0 4px 8px rgba(0,0,0,0.2);
  border-color: var(--link-color);
}

.docs-card h3 {
  margin-top: 0;
  margin-bottom: 0.5rem;
  color: var(--link-color);
  font-size: 1.25rem;
}

.docs-card p {
  margin: 0;
  opacity: 0.9;
}
</style>