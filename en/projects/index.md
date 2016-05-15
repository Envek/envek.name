---
layout: default.en
id: projects
title: 'Novikov Andrey: Projects.'
---

Projects
--------

_In chronological order (from oldest to newest)_

  * ### Zeya.org

    Community portal for the Zeya city (Amur region). I've made the forum and wiki user integration and temperature graph frontend for the values, that are written to database by hardware thermometer, attached to server. *In collaboration with Vladimir Kudashev.*

    URL: [zeya.org](http://zeya.org/)

  * ### “Order manager” for Amur State University

    In the 2nd-3rd grade of university working at software and hardware dept. have written system for managing tasks (orders) for university departments. System is written on Python (Pylons framework) and using PostgreSQL. It's still used at present time.

    URL: [orders.amursu.ru](http://orders.amursu.ru/)

    Source: [GitHub.com/AmurSU/orderman](https://github.com/AmurSU/orderman)

  * ### System for composing the timetable for Amur State University

    Participated in developing and have inherited the system. Have done such things as: timetable editor by groups, main page redesign, teaching plans import from “Plany” software complex. Updating, improving and maintaining system (from server administration to clarification of customer's requirements). Written in Ruby (Ruby on Rails framework) and uses PostgreSQL. *In collaboration with [Andrey Voronkov](https://github.com/Antiarchitect).*

    URL: [taurus.amursu.ru](http://taurus.amursu.ru/)

    Source: [GitHub.com/AmurSU/taurus](https://github.com/AmurSU/taurus)

  * ### Website for Ministry of Economic Development of the Amur region

    HTML coding of already available design. Support. *In collaboration with [Andrey Voronkov](https://github.com/Antiarchitect).*

    Ruby on Rails, MySQL.

    URL: [mer.amurobl.ru](http://mer.amurobl.ru/)

  * ### System for collection and aggregation data of municipal budget programs for Ministry of Economic Development of the Amur region

    I've created aggregation and visualization part (report creation). I've made SVG map of Amur region and integrate it with HTML report via JavaScript. Maintenance, support and improvement. Written on Ruby on Rails and MySQL. *In collaboration with [Andrey Voronkov](https://github.com/Antiarchitect).*

    Example URL: [mcp.amurobl.ru/periods/36/report](http://mcp.amurobl.ru/periods/36/report)

  * ### The system for statistic value grouping for Ministry of Economic Development of the Amur region

    Little ″processing numbers″ webapp. Further development of the svg maps idea. Curently not used.

  * ### Energy Audit

    System for automating collecting data and documentation generation about audit of energy consumption for Amur region Ministry of economic development. Was not launched.

    Ruby on Rails, MySQL.

    URL: [energoaudit.amurobl.ru](http://energoaudit.amurobl.ru/)

  * ### URL-addresses catalogizer for Amur State University

    Simple URL-addresses catalogizer, with correct handling of IDN, user interface for data input, and export to JSON, XML and format, that recognized by Squid proxy server. Written in Ruby on Rails and stores data in PostgreSQL.

    URL: [scatalog.amursu.ru](http://scatalog.amursu.ru/)

    Source code: [GitHub.com/AmurSU/simple_catalog](https://github.com/AmurSU/simple_catalog)

  * ### Amur region ministry of transport website

    Typical website with one feature, that I love. After feedback form submit, a notify email is sent to responsible person, and that person can post a reply to website just by send a reply to this email.

    Ruby on Rails, MySQL.

    URL: [mintrans-amur.ru](http://mintrans-amur.ru/)

  * ### Admission committee software for Amur State University (2013)

    Initially it was just a web form for enrollees to fill application forms, but afterwards whole logic required by committee was implemented: document (application form, contract and more) printing, enrollee logging, enrollee list creation, export info and statistics to XML (for import to 1C:University), PDF (for displaying stats on university TV), and JSON (for import to FIS EGE). This is my first serious system with (relatively) many users, that I've grown myself given only chief’s desire and end users. It is successfully maintained and used after I left the university.

    Ruby on Rails and PostgreSQL.

    URL: [priem.amursu.ru](http://priem.amursu.ru/)

  * ### PostIndexAPI.ru

    System, that illustrates my article [″Using post codes in your application″](http://habrahabr.ru/post/190122/). Using official post codes database by Russian Post with no changes (except for converting to PostgreSQL format). Written in Sinatra.

    URL: [postindexapi.ru](http://postindexapi.ru/)

    Source code: [GitHub.com/Envek/postindexapi.ru](https://github.com/Envek/postindexapi.ru)

  * ### Amur region investment portal

    Typical multilingual website with news feed, tree-like sections structure, WYSIWYG-editor Sphinx-based search, and one feature, that I love: user feedback. After feedback form submit, a notify email is sent to responsible person, and that person can post a reply to website just by send a reply to this email (or via web form). User asked a question is also being informed about reply.

    URL: [invest.amurobl.ru](http://invest.amurobl.ru/)

  * The Official Website of the Russian Ministry of Health (2013-present)

    Rich content web site with modern web UI and integration with various information systems.
    Ruby on Rails based CMS, PostgreSQL, full-text search on Sphinx, Redis for queued jobs, cache, and stats.

    URL: [www.rosminzdrav.ru](https://www.rosminzdrav.ru/)

  * ### Federal Online Registry of the Ministry of Health of the Russian Federation (2014)

    Cloud-based multitenant Federal patient registration and scheduling service featuring AJAX-based web interface for hospital receptionists and medical staff, and universal SOAP and HL7 APIs for diverse healthcare information systems. I joined project on late stage, added support for digital signatures of SOAP messages and did many minor tasks across all apps. Also that was my first meeting with Puppet.

  * ### Regional portal of government and municipal services for Moscow region (2014)

    Ruby on Rails application with a lot of integrations with other systems and data sources such as form player, documents and personal data registry, payment gateway, electronic queue service, and others via JSON, XML-RPC, and SOAP APIs (with digital signatures). Implemented SSO identity provider (OpenID Connect) for sub portals. Project was cancelled right before public launch, but gave me lots of experience! (and burnout)

  * ### Ambulance station automation system (2014–present)

    System for automation of the ambulance station workflow: accepting and dispatching calls, tracking vehicles, calendar planning for entire shift work of station employees, accounting of medicines, fuels, and employees attendance. I participated in development of many parts on the whole application, and also adapted an open source software for receiving data from vehicle trackers written on Erlang, wrote Puppet manifests for rapid deployment, and set up own CA for HTTPS.

    This system is being used in a few of Russian regions.

    Ruby on Rails, PostgreSQL, Angular.js, Websocket.

  * ### System to optimize the logistics of the Sberbank encashment service (2015–present)

    System that automates workflow of bank encashment centers: receives service orders from other systems, generates optimized routes using road network graph and genetic algorithms (our microservice written in Java), manual planning,  order completion with Android app, export of results to other systems. Integrates with a dozen of other systems via MQ, SOAP, and file exchange. I participated in development of one of the backends, integrations, frontend, and deployment via tar archives.

    Ruby on Rails, Oracle 12c, Redis, Ember.js.
