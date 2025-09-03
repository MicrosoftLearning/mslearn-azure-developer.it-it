---
title: Esercizi per sviluppatori di Azure
permalink: index.html
layout: home
---

## Panoramica

Gli esercizi seguenti sono progettati per offrire un'esperienza di apprendimento pratica in cui verranno esaminate le attività comuni eseguite dagli sviluppatori durante la creazione e la distribuzione di soluzioni in Microsoft Azure.

> **Nota**: per completare gli esercizi, è necessaria una sottoscrizione di Azure in cui si dispone di autorizzazioni e quote sufficienti per effettuare il provisioning delle risorse di Azure necessarie. Creazione di un [account di Azure](https://azure.microsoft.com/free), se non già disponibile. 

Alcuni esercizi possono avere requisiti aggiuntivi o diversi. Tali esercizi includeranno una sezione **Prima di iniziare** specifica per ogni esercizio.

## Aree tematiche
{% assign exercises = site.pages | where_exp:"page", "page.url contains '/instructions'" %} {% assign grouped_exercises = exercises | group_by: "lab.topic" %}

<ul>
{% for group in grouped_exercises %}
<li><a href="#{{ group.name | slugify }}">{{ group.name }}</a></li>
{% endfor %}
</ul>

{% for group in grouped_exercises %}

## <a id="{{ group.name | slugify }}"></a>{{ group.name }} 

{% for activity in group.items %} [{{ activity.lab.title }}]({{ site.github.url }}{{ activity.url }}) <br/> {{ activity.lab.description }}

---

{% endfor %} <a href="#overview">Torna all'inizio</a> {% endfor %}

