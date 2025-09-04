---
title: Entwickeln KI-gesteuerter Lösungen für die Informationsextraktion in Azure
permalink: index.html
layout: home
---

Die folgenden Übungen sind darauf ausgelegt, Ihnen eine praktische Lernerfahrung zu bieten, in der Sie allgemeine Aufgaben erkunden, die Entwickler und Entwicklerinnen beim Erstellen von KI-gesteuerten Lösungen für die Informationsextraktion in Microsoft Azure ausführen.

> **Hinweis**: Um die Übungen durchführen zu können, benötigen Sie ein Azure-Abonnement, das Ihnen ausreichende Berechtigungen und Kontingente für die Bereitstellung der erforderlichen Azure-Ressourcen und generativen KI-Modelle bietet. Wenn Sie noch kein Konto haben, können Sie sich für ein [Azure-Konto](https://azure.microsoft.com/free) anmelden. Für neue Benutzende gibt es dort eine kostenlose Testoption, die Guthaben für die ersten 30 Tage beinhaltet.

## Übungen

{% assign labs = site.pages | where_exp:"page", "page.url contains '/Instructions/Labs'" %} {% for activity in labs  %}
<hr>
### [{{ activity.lab.title }}]({{ site.github.url }}{{ activity.url }})

{{activity.lab.description}} {% endfor %}

> **Hinweis**: Diese Übungen können zwar eigenständig absolviert werden, sie sind jedoch als Ergänzung zu den Modulen auf [Microsoft Learn](https://learn.microsoft.com/training/paths/ai-extract-information/) gedacht, in denen Sie tiefer in einige der zugrunde liegenden Konzepte eintauchen können, auf denen diese Übungen basieren.
