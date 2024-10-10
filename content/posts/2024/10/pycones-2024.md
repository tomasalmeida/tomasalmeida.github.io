---
title: "PyConES 2024: my impressions of my first time in a python conference!"
date: 2024-10-06T21:54:45+01:00
draft: false
tags: ["python", "conference"]
categories: ["conferences"]
ShowToc: true
TocOpen: true
weight: 2
---

## PyconEs 2024 in Vigo

PyConES is the annual Python conference in Spain, and it is organized by the Python Spain association. The Python community in Vigo is quite active and so, they were invited to host PyConES 2024. Just to mention too, Python Vigo was  a pillar to the creation of VigoTech Alliance, the meta community of tech communities in Vigo.

Keeping the long story, Confluent was an sponsor (and a colleague had a talk), so I had no excuses to not attend it. If you do not want to hear more, the TL;DR is that I was impressed by the number of people, the quality of the talks and the organization. Congrats to everyone involved! 

## The Agenda and how to choose the talks
![How I feel choosing talks - kid in front of an ice cream shop](kid-icecream.jpeg "How I feel choosing talks - kid in front of an ice cream shop")

You cannot have all flavors, so you have to choose. The agenda was quite complete, with talks for all levels and interests. I was interested in the talks about data governance, data quality and the two related Kafka talks. If these requirements were not met, I tried to choose the odd ones and I believe I did well on that 🙈.

### 0.  Nuestra primera API Restful + auth + OpenAPI

This workshop on Friday was really interesting to understand how REST can be done in Django and Python. I need to review again the well organised repo (https://github.com/APSL/pycones2024) to get a full picture of everything, but it was fun to open the IDE and code a bit (or at least verify what the copilot suggested as a code 😅.

### 1. Keynote

Ismael Faro from IBM gave the keynote and I love his passion about AI and quantum computing. He talked about the future of AI and quantum computing. My key technical points about his talk:
* Learning about SWE-bench and how this AI agents are fixing real bugs in real code.
* InstructLab.ai, a tool that I want to explore and test. If I got it right, it helps to improve the LLM model
* Follow and read M. Andrej Karpathy. A reference in AI that is doing some nice stuff around
* Quantum computing is not a threat to classical computing, but a complement. I liked the analogy of the quantum computer as a coprocessor. CPU will work with GPU (as of today) and with QPUs.

### 2.  Mapeando viviendas turísticas ilegales con Python

Juan told us how he used Python and all data available in the Madrid City Council to map illegal tourist flats. Part of his job can be seen in http://desalkila.madrid/ where you can see difference between the flats with license and the registered ones. As he said, he needs to read a huge mail from the legal team to understand if he can publish his code, but I hope he can do it soon.

### 3. Visualize Realtime Stock Market Data with Kafka and FlinkSQL

Lucia Cerchie, from Confluent, gave a nice talk about how to use Kafka and FlinkSQL to visualize stock market data. I liked the way she explained the architecture and how to use the tools. Sometimes, I forget how impressive is Kafka and Flink combined and the power of the real time processing. I think this blog post is the summary of their talk https://www.confluent.io/blog/how-to-use-flinksql-streamlit-kafka-part-1/. 

### 4. Apache Kafka y Python en Feeberse: Streaming de datos en tiempo real

The folks of Feeberse gave a really dynamic introduction of Kafka in Python with some demos and examples. As they said, be nice and download their software from https://feeberse.com/ and give them feedback. I hope they also take my feedback to start using Confluent Cloud 😅 and Schema Registry.

### 5.   Menos 'hype', más responsabilidad? Quien decide qué en el uso de datos y la IA 

Anna was excellent giving several examples why data governance and AI ethics are important. A talk with several links, reports and phrases that make you think about the future.

### 6.   Representación del Juego del Caos: Convirtiendo secuencias genéticas a imagenes 

Interesting how biology is always a subject that computer engineers love. This talk was about how to convert genetic sequences into images and analyse the image to find patterns.

### 7.  Cómo destruir el mundo usando Python y un virus sintético 

Another talk using biology was a sub-subject. So, creating a virus mixing genes is the base of this talk. I learnt about the crispr-cas technology and how it can be used to create a virus. If you never played the game "plague", I believe that is a good moment to try it.

### 8.   Asegurando la calidad del dato con Databricks 

So, in short, this talk brought me the idea of the six dimensions of data quality. This was really the main thing that I extract from the talk and I will for sure use it in the future.

### 9.   El modelo más importante 

Naomi Ceder, a legend in the Python world, gave a talk about the importance of the model in the software development. Talking how complex the "var" definition can be, she went through the bases of the software development.

### 10.   Para desplegar tengo que coger el coche 

Puga was Puga and again, a talk that make you think... and laugh. The world of the software sometimes forget about the real world and this talk was a good reminder of that.

### 11.    Aprende Aprendizaje por refuerzo jugando a Doom 

You say Doom, you have my attention. This talk was about how to use reinforcement learning to play Doom. I need to check the repos they shared and try to play with it.

## My conclusions

* AI is a hype everywhere, and here is not different.
* Data governance and data quality are important and we need to take care of it.
* Kafka is still this big thing that everyone knows, but not everyone uses.
* Python language is used everywhere and in all kind of projects.
* I love conferences and I need to attend more of them. 

Thanks to all people organising the conference and the speakers. Thank you again Confluent for sponsoring the event and last, but not least, we need to push for more community everywhere.