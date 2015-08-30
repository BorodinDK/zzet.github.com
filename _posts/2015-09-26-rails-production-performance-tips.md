---
layout: post
title: "Ищем, где и почему production приложение тормозит"
date: 2015-09-26 12:15
meta: true
comments: true
categories: ruby rails performance tips production dashboard new_relic skylight profileit profiling
tags: ruby rails performance tips production dashboard new_relic skylight profileit profiling
---

# Ищем, где и почему production приложение тормозит

Многие из разработчиков мира Ruby on Rails знакомы с богатым набором инструментов для профилировали приложения в development среде. Зачастую, работа с этими инструментами заканчивается до/после выкатки фичи в production. Сначала все может работать хорошо и быстро, ну а дальше... как повезет.

В докладе я расскажу о том, как можно в production среде следить за показателями производительности приложений и отлавливать те самые кейсы, когда оно начинает вести себя не так, как хотелось бы.

# О чем я не буду рассказывать

Список инструментов для локального профилирования (рабочая станция разработчика)

# Почему выбирают rails для разработки и как развиваются проекты


# Только хардкор

Перед тем, как говорить о том, как и что искать, необходимо вспомнить, а что у нас вообще есть.

## Из чего состоит application

* Backend codebase
  * Application code
  * Rails code
  * Ruby code
  * Database quries
* Database (databases?)
  * Database quries
* Network
  * Data flow
* External services
  *
* Frontend code
  * DOM
  * JavaScript
  * CSS

# Что значит "application работает быстро"

  * Это ложь
  * Какие метрики существуют

# Как можно следить за производительностью?

* Dashboard
* Alert monitoring system

# На что обращать внимание?

# Какие инструменты есть

* NewRelic
* Skylight
* Profileit.io

# Общие заблуждения и ложные сигналы

# Когда и, что важно, какие решения стоит принимать.

Флоу профилирования приложения

# Lenshq

