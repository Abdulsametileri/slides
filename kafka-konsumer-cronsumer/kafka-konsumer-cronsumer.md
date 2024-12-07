---
marp: true
theme: gaia
transition: fade
author: Abdulsamet İleri
footer: '![w:150 h:150](images/tytech.png)'
style: |
  /* Generic image styling for number icons */
  img:is([alt="1"], [alt="2"], [alt="3"], [alt="4"], [alt="5"]) {
    height: 64px;
    position: relative;
    top: -0.1em;
    vertical-align: middle;
    width: 64px;
  }

  footer {
    height: 150px;
    /* Unset default placing inherited from the built-in theme */
    left: auto;
    right: auto;
    top: auto;
    right: 0px;
    bottom: 0px;
  }
---

# Agenda

![1](https://icongr.am/material/numeric-1-circle.svg?color=666666) Introduction
![2](https://icongr.am/material/numeric-2-circle.svg?color=666666) What is Kafka Cronsumer and Konsumer?
![3](https://icongr.am/material/numeric-3-circle.svg?color=666666) Kafka Konsumer Features with real-life examples
![4](https://icongr.am/material/numeric-4-circle.svg?color=666666) How Kafka Cronsumer is work? What is it's features?
![5](https://icongr.am/material/numeric-5-circle.svg?color=666666) Conclusion

---

<!-- _class: lead -->

![1 w:256 h:256](https://icongr.am/material/numeric-1-circle.svg?color=ff9900)

# Introduction

---

#### Abdulsamet İleri

* Backend dev, +2.5 years on Ty.
* Seller Product Indexing Team _(aggregation domain)_
  * aggregate stock, price, variant, category, listing etc.
  * total 3b content
  * daily 1b invalidations
  * 11 consumers
--- 

<!-- _class: lead -->

![2 w:256 h:256](https://icongr.am/material/numeric-2-circle.svg?color=ff9900)

# What is Kafka Cronsumer and Konsumer?

---

# Kafka C{r}onsumer

* Used for retry/exception strategy management
* Used by `100+` projects in Ty
* Uses [segmentio/kafka-go](https://github.com/segmentio/kafka-go)
* ![cronsumer w:990px](images/cronsumer-hello.png)

<!--
`v1.0.0` Jun 5, 2023, currently `v1.5.5`,  `14` contributors
 -->

---

# Kafka Konsumer

* Kafka consumer library with a built-in retry/exception manager (kafka-cronsumer)
* Used by `100+` projects in Ty
* Uses [segmentio/kafka-go](https://github.com/segmentio/kafka-go)

<!-- 
* `v1.0.0` Jun 5, 2023, currently `v2.4.6`,  `21` contributors
-->

---

![konsumer h:600](images/konsumer-hello.png)

---

![konsumer batch h:600](images/konsumer-batch-hello.png)

---

![konsumer producer h:600](images/konsumer-producer-hello.png)

---

<!-- _class: lead -->

![3 w:256 h:256](https://icongr.am/material/numeric-3-circle.svg?color=ff9900)

# Kafka Konsumer Features with real-life examples

---

### Distributed Tracing Support

* `segmentio/kafka-go` has no opentelemetry support. [otel-kafka-konsumer](https://github.com/Trendyol/otel-kafka-konsumer) born!

<!--otel kafka konsumer'ı dist tracing segmentio ya entegre hali gibi düşünebiliriz. -->

* ![w:900px center](images/dist-tracing.png)

<!--tek satır bir boolean flag vererek, enable edebiliyoruz -->

--- 

# Pause/Resume consumer

* We processed Brand messages in the evening, why?
* ![brand h:310](images/brand.png)

<!--
Sisteme yük oluşturabilecek eventları gece işleme 
1 brand eventi, milyonlarca content'e hit edebilir.
-->

---

![pause resume consumer h:500](images/pause-resume-consumer.png)

---

![pause resume consumer with cron h:500](images/pause-resume-consumer-with-cron.png)

---

# Run pre-batch function for batch mode

- `PreBatch` functions benefits _(same-key compaction, merging, pre-processing)_

<!--
Compaction: same key ile
Merging: Birbiri ile related eventleri birleştirme mesela n listing -> content bazlı birleştirme
pre processing: event ignore, deserialization check
 -->

<!--Use case: category events -->
![](images/same-key-compaction.png)

---

- Before Prebatch
![](images/before-prebatch.png)
<!--
Consumer'ı kapattık
gece işleniyor birde. 
-->

---

- After Prebatch
![](images/after-prebatch.png)

---

- Merging listing events
![width:550px](images/merging_listing_events.png)

---

# Feature: Error-Only Retry for batch consuming

![width:900px](images/erroronlybatch.png)

---

# Feature: Skip Message By Header Feature

![](images/skipbyheader.png)

---

![width:900px](images/skipbyheader-2.png)

---

# Kafka Konsumer Features

- Built-in retry/exception handling support
- High-performance batch-consuming
- Distributed Tracing (Opentelemetry) support
- Pause/Resume consumer any time
- Run pre-batch function for batch mode
- Error-Only Retry for batch consuming
- Skip Message By Header Feature
- Metric collector support (exposed useful metrics)
- Scram — Plain Text Authentication & TLS Support
- Kafka Producer Support

---

# High-Performance batch consuming

| ![](images/batch_supplier.png) | ![](images/batch_reindex.png) |
|---------------------------------|------------------------------|
| ![](images/batch_micro.png)    | ![](images/batch_price.png)  |

<!-- segmentio da batch consumer diye bir şey olmaması. Bizim üzerine bir katman ile yazmamız -->

---

# Behind the scenes: Kafka Cronsumer

| ![](images/listing_exception.png) | ![](images/price_exception.png) |
|---------------------------------|------------------------------|
| ![](images/stock_exception.png)    | ![](images/content_exception.png)  |

---

# Architecture

![](images/cronsumer-architecture.png)

---

# Kafka Cronsumer Features

- Deadletter support
- Backoff strategy support _(linear, fixed, and exponential options)_
- Header filter support
- Metric Collector support
  - kafka_cronsumer_retried_messages_total
  - kafka_cronsumer_discarded_messages_total
- `x-error-message` support 
<!--bir mesajın neden exception topic'e düştüğünü bu header'a bakarak görebiliyoruz -->

You can find a number of ready-to-run examples at [this directory.](https://github.com/Trendyol/kafka-cronsumer/tree/main/examples)

---

# Kafka Cronsumer:  When to use it? 🤔

- Iteration-based back-off strategies are applicable
- Messages could be processed in an eventually consistent state
- Max retry exceeded messages could be ignored
- To increase consumer resiliency
- To increase consumer performance with concurrency

---

# Kafka Cronsumer: When to avoid? ❗️

- Messages should be processed in order
- Messages should be certainly processed _(we discard messages if max retry is exceeded or move to dead letter topic)_
- Messages should be committed _(we use auto-commit interval for increasing performance)_
- Messages with TTL (Time to Live)

---

Thank you to all contributors

| ![](images/konsumer_contribs.png) | ![](images/cronsumer_contribs.png) |
|--------------------------------|-------------------------------|

---

# Conclusion

Thank you for listening

- https://github.com/Trendyol/kafka-konsumer
![](images/konsumer_qr.png)

- https://github.com/Trendyol/kafka-cronsumer
![](images/cronsumer_qr.png)

---

# Contact us

- #kafka-konsumer
- https://pandora.trendyol.com/qa/kafka-konsumer

We are open to hearing your voice and feedback. 
Don’t hesitate to contact us and give ⭐️️ ⭐ ⭐️ 😄

