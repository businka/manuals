Internet Engineering Task Force (IETF)                       E. Rescorla

Request for Comments: 6347                                    RTFM, Inc.

Obsoletes: 4347                                              N. Modadugu

Category: Standards Track                                   Google, Inc.

ISSN: 2070-1721                                             January 2012


#Datagram Transport Layer Security Version 1.2

Abstract

   This document specifies version 1.2 of the Datagram Transport Layer
   Security (DTLS) protocol.  The DTLS protocol provides communications
   privacy for datagram protocols.  The protocol allows client/server
   applications to communicate in a way that is designed to prevent
   eavesdropping, tampering, or message forgery.  The DTLS protocol is
   based on the Transport Layer Security (TLS) protocol and provides
   equivalent security guarantees.  Datagram semantics of the underlying
   transport are preserved by the DTLS protocol.  This document updates
   DTLS 1.0 to work with TLS version 1.2.

Status of This Memo

   This is an Internet Standards Track document.

   This document is a product of the Internet Engineering Task Force
   (IETF).  It represents the consensus of the IETF community.  It has
   received public review and has been approved for publication by the
   Internet Engineering Steering Group (IESG).  Further information on
   Internet Standards is available in Section 2 of RFC 5741.

   Information about the current status of this document, any errata,
   and how to provide feedback on it may be obtained at
   http://www.rfc-editor.org/info/rfc6347.


















Copyright Notice

   Copyright (c) 2012 IETF Trust and the persons identified as the
   document authors.  All rights reserved.

   This document is subject to BCP 78 and the IETF Trust's Legal
   Provisions Relating to IETF Documents
   (http://trustee.ietf.org/license-info) in effect on the date of
   publication of this document.  Please review these documents
   carefully, as they describe your rights and restrictions with respect
   to this document.  Code Components extracted from this document must
   include Simplified BSD License text as described in Section 4.e of
   the Trust Legal Provisions and are provided without warranty as
    described in the Simplified BSD License.

   This document may contain material from IETF Documents or IETF
   Contributions published or made publicly available before November
   10, 2008.  The person(s) controlling the copyright in some of this
   material may not have granted the IETF Trust the right to allow
   modifications of such material outside the IETF Standards Process.
   Without obtaining an adequate license from the person(s) controlling
   the copyright in such materials, this document may not be modified
   outside the IETF Standards Process, and derivative works of it may
   not be created outside the IETF Standards Process, except to format
   it for publication as an RFC or to translate it into languages other
   than English.

# 1.  Introduction

   TLS [TLS] is the most widely deployed protocol for securing network
   traffic.  It is widely used for protecting Web traffic and for e-mail
   protocols such as IMAP [IMAP] and POP [POP].  The primary advantage
   of TLS is that it provides a transparent connection-oriented channel.
   Thus, it is easy to secure an application protocol by inserting TLS
   between the application layer and the transport layer.  However, TLS
   must run over a reliable transport channel -- typically TCP [TCP].
   Therefore, it cannot be used to secure unreliable datagram traffic.

   An increasing number of application layer protocols have been
   designed that use UDP transport.  In particular, protocols such as
   the Session Initiation Protocol (SIP) [SIP] and electronic gaming
   protocols are increasingly popular.  (Note that SIP can run over both
   TCP and UDP, but that there are situations in which UDP is
   preferable.)  Currently, designers of these applications are faced
   with a number of unsatisfactory choices.  First, they can use IPsec
   [RFC4301].  However, for a number of reasons detailed in [WHYIPSEC],
   this is only suitable for some applications.  Second, they can design
   a custom application layer security protocol.  Unfortunately,
   although application layer security protocols generally provide
   superior security properties (e.g., end-to-end security in the case
   of S/MIME), they typically require a large amount of effort to design
   -- in contrast to the relatively small amount of effort required to
   run the protocol over TLS.

   In many cases, the most desirable way to secure client/server
   applications would be to use TLS; however, the requirement for
   datagram semantics automatically prohibits use of TLS.  This memo
   describes a protocol for this purpose: Datagram Transport Layer
   Security (DTLS).  DTLS is deliberately designed to be as similar to
   TLS as possible, both to minimize new security invention and to
   maximize the amount of code and infrastructure reuse.

   DTLS 1.0 [DTLS1] was originally defined as a delta from [TLS11].
   This document introduces a new version of DTLS, DTLS 1.2, which is
   defined as a series of deltas to TLS 1.2 [TLS12].  There is no DTLS
   1.1; that version number was skipped in order to harmonize version
   numbers with TLS.  This version also clarifies some confusing points
   in the DTLS 1.0 specification.

   Implementations that speak both DTLS 1.2 and DTLS 1.0 can
   interoperate with those that speak only DTLS 1.0 (using DTLS 1.0 of
   course), just as TLS 1.2 implementations can interoperate with
   previous versions of TLS (see Appendix E.1 of [TLS12] for details),
   with the exception that there is no DTLS version of SSLv2 or SSLv3,
   so backward compatibility issues for those protocols do not apply.

## 1.1. Requirements Terminology

   The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT",
   "SHOULD", "SHOULD NOT", "RECOMMENDED", "MAY", and "OPTIONAL" in this
   document are to be interpreted as described in RFC 2119 [REQ].

# 2.  Usage Model

   The DTLS protocol is designed to secure data between communicating
   applications.  It is designed to run in application space, without
   requiring any kernel modifications.

   Datagram transport does not require or provide reliable or in-order
   delivery of data.  The DTLS protocol preserves this property for
   payload data.  Applications such as media streaming, Internet
   telephony, and online gaming use datagram transport for communication
   due to the delay-sensitive nature of transported data.  The behavior
   of such applications is unchanged when the DTLS protocol is used to
   secure communication, since the DTLS protocol does not compensate for
   lost or re-ordered data traffic.

# 3. Обзор DTLS

Основная философия проектирования DTLS заключается в построении «TLS поверх передачи дейтаграмм». Причина того, 
что TLS не может использоваться непосредственно в среде дейтаграмм, заключается просто в том, что пакеты могут 
быть потеряны или переупорядочены. TLS не имеет внутренних средств для обработки такого рода ненадежности; 
поэтому реализации TLS ломаются при повторном размещении на транспорте дейтаграмм. Цель DTLS - внести в TLS 
только минимальные изменения, необходимые для решения этой проблемы. В максимально возможной степени DTLS 
идентичен TLS. Всякий раз, когда нам нужно изобрести новые механизмы, мы стараемся делать это таким образом, 
чтобы сохранить стиль TLS.

Ненадежность создает проблемы для TLS на двух уровнях:

1. TLS не допускает независимого дешифрования отдельных записей. Поскольку проверка целостности зависит от 
порядкового номера, если запись N не получена, то проверка целостности записи N + 1 будет основана на неправильном 
порядковом номере и, таким образом, не удастся. (Обратите внимание, что до TLS 1.1 не было явного IV, 
поэтому расшифровка также не удалась.)

2. Уровень подтверждения TLS предполагает, что сообщения подтверждения доставляются надежно, и прерывается, 
   если эти сообщения потеряны.
   

Остальная часть этого раздела описывает подход, который DTLS использует для решения этих проблем.


## 3.1. Обмен сообщениями без потерь

На уровне шифрования трафика TLS (называемом уровнем записи TLS) записи не являются независимыми. Есть два вида 
зависимости между записями:

1. Криптографический контекст (поток ключей потокового шифра) сохраняется между записями.

2. Защита от повторного воспроизведения и переупорядочения сообщений обеспечивается MAC, который включает 
   порядковый номер, но порядковые номера неявно присутствуют в записях.

DTLS решает первую проблему, запрещая потоковые шифры. DTLS решает вторую проблему, добавляя явные порядковые номера.

## 3.2. Обеспечение надежности рукопожатия

Рукопожатие TLS - это шифрованное рукопожатие с блокировкой. Сообщения должны передаваться и приниматься в 
определенном порядке; любой другой заказ является ошибкой. Ясно, что это несовместимо с переупорядочиванием и 
потерей сообщений. Кроме того, сообщения подтверждения TLS потенциально больше, чем любая данная дейтаграмма, 
что создает проблему фрагментации IP. DTLS должен предоставлять исправления для обеих этих проблем.

### 3.2.1. Потеря пакетов

DTLS использует простой таймер повторной передачи для обработки потери пакетов. На следующем рисунке показана 
основная концепция с использованием первой фазы рукопожатия DTLS:   

         Client                                   Server
         ------                                   ------
         ClientHello           ------>
    
                                 X<-- HelloVerifyRequest
                                                  (lost)
    
         [Timer Expires]
    
         ClientHello           ------>
         (retransmit)

После того, как клиент передал сообщение ClientHello, он ожидает увидеть HelloVerifyRequest от сервера. Однако, 
если сообщение сервера потеряно, клиент знает, что либо ClientHello, либо HelloVerifyRequest было потеряно, и 
передает его повторно. Когда сервер получает повторную передачу, он знает, что нужно повторить передачу.

Сервер также поддерживает таймер повторной передачи и выполняет повторную передачу по истечении этого таймера.

Обратите внимание, что тайм-аут и повторная передача не применяются к HelloVerifyRequest, потому что это потребует 
создания состояния на сервере. HelloVerifyRequest спроектирован так, чтобы быть достаточно маленьким, чтобы он сам 
не был фрагментирован, что позволяет избежать проблем, связанных с чередованием нескольких запросов HelloVerifyRequest.

### 3.2.2. Переупорядочивание

В DTLS каждому сообщению квитирования назначается определенный порядковый номер в этом квитировании. Когда 
одноранговый узел получает сообщение подтверждения, он может быстро определить, является ли это сообщение 
следующим ожидаемым им сообщением. Если да, то обрабатывает. В противном случае он ставит его в очередь для 
будущей обработки после получения всех предыдущих сообщений.

### 3.2.3. Размер сообщения

Сообщения подтверждения TLS и DTLS могут быть довольно большими (теоретически до 2 ^ 24-1 байтов, на практике - 
до многих килобайт). Напротив, дейтаграммы UDP часто ограничиваются размером <1500 байтов, если фрагментация IP 
нежелательна. Чтобы компенсировать это ограничение, каждое сообщение подтверждения DTLS может быть фрагментировано 
по нескольким записям DTLS, каждая из которых предназначена для размещения в одной дейтаграмме IP. Каждое сообщение 
подтверждения DTLS содержит как смещение фрагмента, так и длину фрагмента. Таким образом, получатель, владеющий 
всеми байтами сообщения подтверждения, может повторно собрать исходное нефрагментированное сообщение.

## 3.3. Обнаружение повтора

DTLS дополнительно поддерживает обнаружение воспроизведения записи. Используемый метод такой же, как и в IPsec AH / ESP, 
за счет поддержки окна битовой карты полученных записей. Записи, которые слишком старые, чтобы уместиться в окне, и 
записи, которые были получены ранее, автоматически отбрасываются. Функция обнаружения повторного воспроизведения 
является необязательной, поскольку дублирование пакетов не всегда является вредоносным, но также может возникать 
из-за ошибок маршрутизации. Возможно, приложения могут обнаруживать повторяющиеся пакеты и соответственно изменять 
свою стратегию передачи данных.

# 4. Отличия от TLS

Как упоминалось в разделе 3, DTLS намеренно очень похож на TLS. Поэтому вместо того, чтобы представлять DTLS
как новый протокол, мы представляем его как серию дельт от TLS 1.2 [TLS12]. Там, где мы явно не называем различия, 
DTLS такой же, как в [TLS12].

## 4.1. Слой записи

Уровень записи DTLS очень похож на слой TLS 1.2. Единственное изменение - это включение в запись явного порядкового
номера. Этот порядковый номер позволяет получателю правильно проверить MAC-адрес TLS. Формат записи DTLS показан ниже:

      struct {
           ContentType type;
           ProtocolVersion version;
           uint16 epoch;                                    // New field
           uint48 sequence_number;                          // New field
           uint16 length;
           opaque fragment[DTLSPlaintext.length];
         } DTLSPlaintext;

   type
       Эквивалентно полю типа в записи TLS 1.2.

   version
        Версия используемого протокола. В этом документе описывается версия 1.2 DTLS, в которой используется
        версия {254, 253}. Значение версии 254.253 является дополнением до 1 DTLS версии 1.2. Этот максимальный
        интервал между номерами версий TLS и DTLS гарантирует, что записи из двух протоколов могут быть легко
        различимы. Следует отметить, что будущие оперативные номера версий DTLS будут уменьшаться в значении 
        (в то время как истинный номер версии увеличивается в значении).

   epoch
        Значение счетчика, которое увеличивается при каждом изменении состояния шифра.

   sequence_number
      Порядковый номер этой записи.

   length
      Идентично полю длины в записи TLS 1.2. Как и в TLS 1.2, длина не должна превышать 2 ^ 14.

   fragment
      Идентично полю фрагмента записи TLS 1.2.

DTLS использует явный порядковый номер, а не неявный, передаваемый в поле sequence_number записи. Порядковые номера  
поддерживаются отдельно для каждой эпохи, причем каждый sequence_number изначально равен 0 для каждой эпохи. Например,
если сообщение квитирования из эпохи 0 передается повторно, оно может иметь порядковый номер после сообщения
из эпохи 1, даже если сообщение из эпохи 1 было передано первым. Обратите внимание, что во время рукопожатия
необходимо проявлять некоторую осторожность, чтобы гарантировать, что повторно передаваемые сообщения используют 
правильную эпоху и ключевой материал.

Если несколько рукопожатий выполняются в близкой последовательности, на проводе может быть несколько записей с одним 
и тем же порядковым номером, но из разных состояний шифра. Поле эпохи позволяет получателям различать такие пакеты.
Номер эпохи изначально равен нулю и увеличивается каждый раз при отправке сообщения ChangeCipherSpec. Чтобы 
гарантировать, что любая заданная пара последовательность / эпоха уникальна, реализации НЕ ДОЛЖНЫ допускать повторное 
использование одного и того же значения эпохи в течение двухкратного максимального срока жизни сегмента TCP. 
На практике реализации TLS редко меняются; поэтому мы не ожидаем, что это будет проблемой.

   Note that because DTLS records may be reordered, a record from epoch
   1 may be received after epoch 2 has begun.  In general,
   implementations SHOULD discard packets from earlier epochs, but if
   packet loss causes noticeable problems they MAY choose to retain
   keying material from previous epochs for up to the default MSL
   specified for TCP [TCP] to allow for packet reordering.  (Note that
   the intention here is that implementors use the current guidance from
   the IETF for MSL, not that they attempt to interrogate the MSL that
   the system TCP stack is using.)  Until the handshake has completed,
   implementations MUST accept packets from the old epoch.

   Conversely, it is possible for records that are protected by the
   newly negotiated context to be received prior to the completion of a
   handshake.  For instance, the server may send its Finished message
   and then start transmitting data.  Implementations MAY either buffer
   or discard such packets, though when DTLS is used over reliable
   transports (e.g., SCTP), they SHOULD be buffered and processed once
   the handshake completes.  Note that TLS's restrictions on when
   packets may be sent still apply, and the receiver treats the packets
   as if they were sent in the right order.  In particular, it is still
   impermissible to send data prior to completion of the first
   handshake.

   Note that in the special case of a rehandshake on an existing
   association, it is safe to process a data packet immediately, even if
   the ChangeCipherSpec or Finished messages have not yet been received
   provided that either the rehandshake resumes the existing session or
   that it uses exactly the same security parameters as the existing
   association.  In any other case, the implementation MUST wait for the
   receipt of the Finished message to prevent downgrade attack.

   As in TLS, implementations MUST either abandon an association or
   rehandshake prior to allowing the sequence number to wrap.
   Similarly, implementations MUST NOT allow the epoch to wrap, but
   instead MUST establish a new association, terminating the old
   association as described in Section 4.2.8.  In practice,
   implementations rarely rehandshake repeatedly on the same channel, so
   this is not likely to be an issue.

### 4.1.1. Отображение транспортного уровня

Каждая запись DTLS ДОЛЖНА помещаться в одну дейтаграмму. Чтобы избежать фрагментации IP, клиентам уровня записи
DTLS СЛЕДУЕТ попытаться изменить размер записей так, чтобы они соответствовали любым оценкам PMTU, полученным 
из уровня записи.

Обратите внимание, что в отличие от IPsec, записи DTLS не содержат идентификаторов ассоциации. Приложения должны 
быть организованы для мультиплексирования между ассоциациями. С UDP это, по-видимому, делается с помощью 
номера хоста / порта.

Несколько записей DTLS могут быть помещены в одну дейтаграмму. Они просто кодируются последовательно. 
Для определения границ достаточно кадрирования записи DTLS. Обратите внимание, однако, что первый байт полезной 
нагрузки дейтаграммы должен быть началом записи. Записи не могут охватывать дейтаграммы.

Некоторые транспорты, такие как DCCP [DCCP], предоставляют свои собственные порядковые номера. При переносе 
через эти транспорты будут присутствовать как DTLS, так и порядковые номера транспорта. Хотя это приводит к 
небольшой неэффективности, порядковые номера транспортного уровня и DTLS служат разным целям; поэтому для 
концептуальной простоты лучше использовать оба порядковых номера. В будущем могут быть указаны расширения DTLS, 
которые позволят использовать только один набор порядковых номеров для развертывания в ограниченных средах.

Некоторые транспортные средства, такие как DCCP, обеспечивают контроль перегрузки передаваемого по ним трафика. 
Если окно перегрузки достаточно узкое, повторные передачи установления связи DTLS могут удерживаться, 
а не передаваться немедленно, что может привести к тайм-аутам и ложной повторной передаче. Когда DTLS используется 
для таких транспортов, следует проявлять осторожность, чтобы не выйти за пределы вероятного окна перегрузки. 
[DCCPDTLS] определяет отображение DTLS в DCCP, которое учитывает эти проблемы.

#### 4.1.1.1. Проблемы PMTU

В общем, философия DTLS заключается в том, чтобы предоставить обнаружение PMTU приложению. Однако DTLS не может 
полностью игнорировать PMTU по трем причинам:

* Кадрирование записи DTLS увеличивает размер дейтаграммы, тем самым снижая эффективный PMTU с точки зрения приложения.

* В некоторых реализациях приложение может не связываться напрямую с сетью, и в этом случае стек DTLS может поглощать 
  индикацию ICMP [RFC1191] «Датаграмма слишком большая» или ICMPv6 [RFC4443] «Слишком большой пакет».

* Сообщения подтверждения DTLS могут превышать PMTU.

Чтобы справиться с первыми двумя проблемами, уровень записи DTLS ДОЛЖЕН вести себя, как описано ниже.

Если оценки PMTU доступны из нижележащего транспортного протокола, они должны быть доступны протоколам верхнего уровня.
 Особенно:
* Для DTLS через UDP протоколу верхнего уровня СЛЕДУЕТ разрешить получать оценку PMTU, поддерживаемую на уровне IP.
* Для DTLS через DCCP протоколу верхнего уровня СЛЕДУЕТ разрешить получать текущую оценку PMTU.
* Для DTLS через TCP или SCTP, которые автоматически фрагментируют и собирают дейтаграммы, ограничения PMTU отсутствуют. 
  Однако протокол верхнего уровня НЕ ДОЛЖЕН записывать записи, размер которых превышает максимальный размер 
  записи 2 ^ 14 байтов.

Уровень записи DTLS ДОЛЖЕН позволять протоколу верхнего уровня обнаруживать величину расширения записи, ожидаемую 
обработкой DTLS. Обратите внимание, что это число является приблизительным из-за заполнения блока и потенциального 
использования сжатия DTLS.

Если имеется индикация транспортного протокола (либо через ICMP, либо через отказ в отправке дейтаграммы, как в 
разделе 14 [DCCP]), то уровень записи DTLS ДОЛЖЕН сообщить протоколу верхнего уровня об ошибке.

Уровень записи DTLS НЕ ДОЛЖЕН мешать протоколам верхнего уровня, выполняющим обнаружение PMTU, через механизмы 
[RFC1191] или [RFC4821]. Особенно:

* Если это разрешено нижележащим транспортным протоколом, протоколу верхнего уровня СЛЕДУЕТ разрешить устанавливать 
  состояние бита DF (в IPv4) или запрещать локальную фрагментацию (в IPv6).

* Если базовый транспортный протокол позволяет приложению запрашивать зондирование PMTU (например, DCCP), уровень 
  записи DTLS должен учитывать этот запрос.

Последний вопрос - протокол установления связи DTLS. С точки зрения уровня записи DTLS, это просто еще один протокол 
верхнего уровня. Однако рукопожатия DTLS происходят нечасто и включают лишь несколько циклов приема-передачи; 
следовательно, обработка PMTU протокола установления связи дает преимущество быстрому завершению, а не точному 
обнаружению PMTU. Чтобы разрешить соединения в этих обстоятельствах, реализации DTLS ДОЛЖНЫ следовать следующим 
правилам:

* Если уровень записи DTLS сообщает уровню подтверждения DTLS, что сообщение слишком велико, он ДОЛЖЕН немедленно 
попытаться фрагментировать его, используя любую существующую информацию о PMTU.

* Если повторные повторные передачи не приводят к ответу, а PMTU неизвестен, последующие повторные передачи ДОЛЖНЫ 
  вернуться к меньшему размеру записи, фрагментируя сообщение квитирования соответствующим образом. Этот стандарт не 
  определяет точное количество попыток повторной передачи перед тем, как отступить, но 2-3 кажется подходящим.
  
### 4.1.2. Защита от записи

Как и TLS, DTLS передает данные как серию защищенных записей. Остальная часть этого раздела описывает 
детали этого формата.

#### 4.1.2.1. MAC

MAC-адрес DTLS такой же, как и у TLS 1.2. Однако вместо использования неявного порядкового номера TLS порядковый 
номер, используемый для вычисления MAC, представляет собой 64-битное значение, сформированное путем конкатенации 
эпохи и порядкового номера в том порядке, в котором они появляются в сети. Обратите внимание, что 
номер эпохи + DTLS имеет ту же длину, что и порядковый номер TLS.

Вычисление TLS MAC параметризуется номером версии протокола, который в случае DTLS является беспроводной версией, 
т.Е. {254, 253} для DTLS 1.2.

Обратите внимание, что одно важное различие между обработкой MAC-адресов DTLS и TLS заключается в том, что в 
TLS ошибки MAC-адреса должны приводить к разрыву соединения. В DTLS принимающая реализация МОЖЕТ просто отбросить
некорректную запись и продолжить соединение. Это изменение возможно, потому что записи DTLS не зависят друг от
друга так, как записи TLS.

В общем, реализациям DTLS СЛЕДУЕТ незаметно отбрасывать записи с неверными MAC или иным образом недействительными. 
Они МОГУТ зарегистрировать ошибку. Если реализация DTLS решает генерировать предупреждение при получении сообщения 
с недопустимым MAC-адресом, она ДОЛЖНА генерировать предупреждение bad_record_mac с фатальным уровнем и завершать 
свое состояние соединения. Обратите внимание: поскольку ошибки не приводят к разрыву соединения, стеки DTLS 
являются более эффективными оракулами типов ошибок, чем стеки TLS. Таким образом, особенно важно следовать 
советам из Раздела 6.2.3.2 [TLS12].

#### 4.1.2.2. Нулевой или стандартный потоковый шифр

Шифр DTLS NULL выполняется точно так же, как шифр TLS 1.2 NULL.

Единственный потоковый шифр, описанный в TLS 1.2, - это RC4, к которому нельзя получить произвольный доступ. 
RC4 НЕ ДОЛЖЕН использоваться с DTLS.

#### 4.1.2.3. Блочный шифр

Шифрование и дешифрование блочного шифра DTLS выполняется точно так же, как в TLS 1.2.

#### 4.1.2.4. Шифры AEAD

TLS 1.2 представил аутентифицированное шифрование с дополнительными наборами шифров (AEAD). Существующие наборы 
шифров AEAD, определенные в [ECCGCM] и [RSAGCM], могут использоваться с DTLS точно так же, как с TLS 1.2.

#### 4.1.2.5. Новые наборы шифров

После регистрации новые комплекты шифров TLS ДОЛЖНЫ указывать, подходят ли они для использования DTLS, и какие 
изменения, если таковые имеются, должны быть выполнены (см. Раздел 7 для рассмотрения IANA).

#### 4.1.2.6. Анти-повтор

Записи DTLS содержат порядковый номер для обеспечения защиты от воспроизведения. Проверка порядкового номера ДОЛЖНА 
выполняться с использованием следующей процедуры скользящего окна, заимствованной из раздела 3.4.3 [ESP].

Счетчик пакетов приемника для этого сеанса ДОЛЖЕН быть инициализирован до нуля, когда сеанс установлен. Для каждой 
полученной записи получатель ДОЛЖЕН проверить, что запись содержит порядковый номер, который не дублирует порядковый 
номер любой другой записи, полученной в течение этого сеанса. Это ДОЛЖНО быть первой проверкой, применяемой к 
пакету после того, как он был сопоставлен с сеансом, чтобы ускорить отклонение повторяющихся записей.

Дубликаты отклоняются с помощью скользящего окна приема. (Каким образом реализовано окно - это вопрос местного 
значения, но следующий текст описывает функциональные возможности, которые должна демонстрировать реализация.) ДОЛЖЕН 
поддерживаться минимальный размер окна 32, но размер окна 64 является предпочтительным, и его СЛЕДУЕТ использовать 
по умолчанию. . Другой размер окна (больше минимального) МОЖЕТ быть выбран получателем. (Получатель не уведомляет 
отправителя о размере окна.)

«Правый» край окна представляет собой наивысшее подтвержденное значение порядкового номера, полученное в этом сеансе. 
Записи, содержащие порядковые номера ниже «левого» края окна, отклоняются. Пакеты, попадающие в окно, проверяются по 
списку полученных пакетов в окне. Эффективные средства для выполнения этой проверки, основанные на использовании 
битовой маски, описаны в разделе 3.4.3 [ESP].

Если полученная запись попадает в окно и является новой, или если пакет находится справа от окна, то получатель 
переходит к проверке MAC. Если проверка MAC не удалась, получатель ДОЛЖЕН отбросить полученную запись как 
недействительную. Окно приема обновляется, только если проверка MAC прошла успешно.

#### 4.1.2.7. Обработка недействительных записей

   В отличие от TLS, DTLS устойчив к недопустимым записям (например, недопустимому форматированию, длине, MAC и т. Д.). В общем, недопустимые записи ДОЛЖНЫ быть отброшены без уведомления, таким образом сохраняя ассоциацию; тем не менее, ошибка МОЖЕТ регистрироваться в целях диагностики. Реализации, которые предпочитают генерировать оповещение, ДОЛЖНЫ генерировать оповещения фатального уровня, чтобы избежать атак, когда злоумышленник неоднократно проверяет реализацию, чтобы увидеть, как она реагирует на различные типы ошибок. Обратите внимание, что если DTLS запускается через UDP, то любая реализация, которая делает это, будет чрезвычайно восприимчива к атакам типа «отказ в обслуживании» (DoS), потому что подделка UDP очень проста. Таким образом, такая практика НЕ ​​РЕКОМЕНДУЕТСЯ для таких перевозок.

   Если DTLS передается через транспорт, устойчивый к подделке (например, SCTP с SCTP-AUTH), то безопаснее отправлять предупреждения, потому что у злоумышленника возникнут трудности с подделкой дейтаграммы, которая не будет отклонена транспортным уровнем.
## 4.2. Протокол установления связи DTLS

DTLS использует все те же сообщения и потоки рукопожатия, что и TLS, с тремя принципиальными изменениями:

1. Для предотвращения отказа был добавлен обмен файлами cookie без сохранения состояния.
    служебные атаки.
2. Модификации заголовка подтверждения для обработки потери сообщения, переупорядочения и фрагментации сообщения DTLS 
(во избежание фрагментации IP).

3. Таймеры повторной передачи для обработки потери сообщения.

За этими исключениями форматы, потоки и логика сообщений DTLS такие же, как и в TLS 1.2.

### 4.2.1. Меры противодействия отказу в обслуживании

Протоколы безопасности дейтаграмм чрезвычайно восприимчивы к различным DoS-атакам. Особое
беспокойство вызывают две атаки:

1. Злоумышленник может потреблять чрезмерные ресурсы на сервере, передавая серию запросов
на установление связи, в результате чего сервер выделяет состояние и потенциально может
выполнять дорогостоящие криптографические операции.

2. Злоумышленник может использовать сервер в качестве усилителя, отправив сообщения об
установлении соединения с поддельным источником жертвы. Затем сервер отправляет свое
следующее сообщение (в DTLS - сообщение сертификата, которое может быть довольно большим)
на машину жертвы, тем самым переполняя ее.

Чтобы противостоять обеим этим атакам, DTLS заимствует технологию cookie без сохранения
состояния, используемую Photuris [PHOTURIS] и IKE IKEv2]. Когда клиент отправляет сообщение
ClientHello серверу, сервер МОЖЕТ ответить сообщением HelloVerifyRequest. Это сообщение
содержит cookie без сохранения состояния, созданный с использованием технологии [PHOTURIS].
Клиент ДОЛЖЕН повторно передать ClientHello с добавленным файлом cookie. Затем сервер
проверяет файл cookie и переходит к рукопожатию, только если он действителен. Этот 
механизм заставляет злоумышленника / клиента получать cookie, что затрудняет DoS-атаки
с поддельными IP-адресами. Этот механизм не обеспечивает никакой защиты от DoS-атак,
установленных с действительных IP-адресов.

Обмен показан ниже:

      Client                                   Server
      ------                                   ------
      ClientHello           ------>
    
                            <-----        HelloVerifyRequest
                                          (contains cookie)
    
      ClientHello           ------>
      (with cookie)
    
      [Rest of handshake]

Поэтому DTLS изменяет сообщение ClientHello, добавляя значение cookie.

       struct {
         ProtocolVersion client_version;
         Random random;
         SessionID session_id;
         opaque cookie<0..2^8-1>;                             // New field
         CipherSuite cipher_suites<2..2^16-1>;
               CompressionMethod compression_methods<1..2^8-1>;
       } ClientHello;

При отправке первого ClientHello у клиента еще нет cookie; в этом случае поле Cookie
остается пустым (нулевая длина).

Определение HelloVerifyRequest выглядит следующим образом:

       struct {
         ProtocolVersion server_version;
         opaque cookie<0..2^8-1>;
       } HelloVerifyRequest;

Тип сообщения HelloVerifyRequest - hello_verify_request (3).

Поле server_version имеет тот же синтаксис, что и в TLS. Однако, чтобы избежать
требования выполнять согласование версии при начальном рукопожатии, реализации
сервера DTLS 1.2 ДОЛЖНЫ использовать DTLS версии 1.0 независимо от версии TLS,
согласование которой ожидается. Клиенты DTLS 1.2 и 1.0 ДОЛЖНЫ использовать версию
исключительно для указания форматирования пакета (которое одинаково в DTLS 1.2 и 1.0),
а не как часть согласования версии. В частности, клиенты DTLS 1.2 НЕ ДОЛЖНЫ предполагать,
что из-за того, что сервер использует версию 1.0 в HelloVerifyRequest, сервер не 
является DTLS 1.2 или что он в конечном итоге согласует DTLS 1.0, а не DTLS 1.2.


При ответе на HelloVerifyRequest клиент ДОЛЖЕН использовать те же значения
параметров (версия, random, session_id, cipher_suites, compress_method), 
что и в исходном ClientHello. Серверу СЛЕДУЕТ использовать эти значения для
создания своего файла cookie и проверки их правильности после получения файла cookie.
Сервер ДОЛЖЕН использовать тот же номер версии в HelloVerifyRequest,
что и при отправке ServerHello. После получения ServerHello клиент ДОЛЖЕН убедиться,
что значения версии сервера совпадают. Чтобы избежать дублирования порядковых
номеров в случае нескольких HelloVerifyRequests, сервер ДОЛЖЕН использовать
порядковый номер записи в ClientHello в качестве порядкового номера записи
в HelloVerifyRequest.

Примечание. Эта спецификация увеличивает ограничение размера файлов cookie
до 255 байтов для большей гибкости в будущем. Предел остается 32 для
предыдущих версий DTLS.

Сервер DTLS ДОЛЖЕН генерировать файлы cookie таким образом, чтобы их можно
было проверить без сохранения состояния каждого клиента на сервере. Один из
способов состоит в том, чтобы иметь случайно сгенерированный секрет и
генерировать файлы cookie как:

      Cookie = HMAC(Secret, Client-IP, Client-Parameters)

Когда получен второй ClientHello, сервер может проверить, что Cookie действителен и что клиент может
получать пакеты по заданному IP-адресу. Чтобы избежать дублирования порядковых номеров в случае обмена
несколькими файлами cookie, сервер ДОЛЖЕН использовать порядковый номер записи в ClientHello в качестве
порядкового номера записи в своем начальном ServerHello. Последующие ServerHellos будут отправлены только
после того, как сервер создаст состояние, и ДОЛЖНЫ нормально увеличиваться.

Одна из возможных атак на эту схему состоит в том, что злоумышленник собирает несколько файлов cookie
с разных адресов и затем повторно использует их для атаки на сервер. Сервер может защититься от этой атаки,
часто изменяя значение секрета, тем самым делая эти файлы cookie недействительными. Если сервер желает,
чтобы легитимные клиенты могли установить связь через переход (например, они получили cookie с секретом 1,
а затем отправили второй ClientHello после того, как сервер перешел на секрет 2), сервер может иметь
ограниченное окно, в течение которого он принимает оба секрета. [IKEv2] предлагает добавить в файлы cookie
номер версии, чтобы обнаружить этот случай. Альтернативный подход - просто попытаться проверить оба секрета.

Серверы DTLS ДОЛЖНЫ выполнять обмен файлами cookie всякий раз, когда выполняется новое рукопожатие.
Если сервер работает в среде, где усиление не является проблемой, сервер МОЖЕТ быть настроен так,
чтобы не выполнять обмен файлами cookie. Однако по умолчанию СЛЕДУЕТ выполнять обмен. Кроме того,
сервер МОЖЕТ отказаться от обмена файлами cookie при возобновлении сеанса. Клиенты ДОЛЖНЫ быть готовы
к обмену файлами cookie при каждом рукопожатии.

Если используется HelloVerifyRequest, начальные ClientHello и HelloVerifyRequest не включаются в расчет
handshake_messages (для сообщения CertificateVerify) и verify_data (для сообщения Finished).

Если сервер получает сообщение ClientHello с недопустимым файлом cookie, он ДОЛЖЕН рассматривать его так же,
как ClientHello без файла cookie. Это позволяет избежать состояний гонки / взаимоблокировки, если клиент
 каким-то образом получает плохой файл cookie (например, из-за того, что сервер меняет свой ключ подписи cookie).

Примечание для разработчиков: это может привести к тому, что клиенты получат несколько сообщений HelloVerifyRequest
с разными файлами cookie. Клиенты ДОЛЖНЫ обработать это, отправив новый ClientHello с файлом cookie
в ответ на новый HelloVerifyRequest.

### 4.2.2. Формат сообщения рукопожатия

Чтобы поддерживать потерю сообщений, переупорядочивание и фрагментацию сообщений,
DTLS изменяет заголовок рукопожатия TLS 1.2:

       struct {
         HandshakeType msg_type;
         uint24 length;
         uint16 message_seq;                               // New field
         uint24 fragment_offset;                           // New field
         uint24 fragment_length;                           // New field
         select (HandshakeType) {
           case hello_request: HelloRequest;
           case client_hello:  ClientHello;
           case hello_verify_request: HelloVerifyRequest;  // New type
           case server_hello:  ServerHello;
           case certificate:Certificate;
           case server_key_exchange: ServerKeyExchange;
           case certificate_request: CertificateRequest;
           case server_hello_done:ServerHelloDone;
           case certificate_verify:  CertificateVerify;
           case client_key_exchange: ClientKeyExchange;
           case finished: Finished;
         } body;
       } Handshake;

Первое сообщение, которое каждая сторона передает в каждом рукопожатии, всегда имеет message_seq = 0.
Каждый раз, когда генерируется каждое новое сообщение, значение message_seq увеличивается на единицу.
Обратите внимание, что в случае повторного встряхивания это означает, что HelloRequest будет иметь message_seq = 0,
а ServerHello будет иметь message_seq = 1. При повторной передаче сообщения используется то же значение message_seq. 
Например:

         Client                             Server
         ------                             ------
         ClientHello (seq=0)  ------>
    
                                 X<-- HelloVerifyRequest (seq=0)
                                                 (lost)
    
         [Timer Expires]
    
         ClientHello (seq=0)  ------>
         (retransmit)
    
                              <------ HelloVerifyRequest (seq=0)
    
         ClientHello (seq=1)  ------>
         (with cookie)
    
                              <------        ServerHello (seq=1)
                              <------        Certificate (seq=2)
                              <------    ServerHelloDone (seq=3)
    
         [Rest of handshake]

Однако обратите внимание, что с точки зрения уровня записи DTLS повторная передача является
новой записью. Эта запись будет иметь новое значение DTLSPlaintext.sequence_number.

Реализации DTLS поддерживают (по крайней мере, условно) счетчик next_receive_seq. Этот счетчик изначально
установлен на ноль.    Когда сообщение получено, если его порядковый номер совпадает с next_receive_seq, 
next_receive_seq увеличивается на единицу, и сообщение обрабатывается. Если порядковый номер меньше
next_receive_seq, сообщение ДОЛЖНО быть отброшено. Если порядковый номер больше next_receive_seq,
реализация ДОЛЖНА поставить сообщение в очередь, но МОЖЕТ его отбросить. (Это простой компромисс между 
пространством и пропускной способностью).

### 4.2.3. Фрагментация и повторная сборка сообщения рукопожатия

Как отмечено в разделе 4.1.1, каждое сообщение DTLS ДОЛЖНО соответствовать одной дейтаграмме транспортного уровня. 
Однако сообщения о подтверждении потенциально превышают максимальный размер записи. Таким образом, DTLS 
предоставляет механизм для фрагментации сообщения подтверждения по ряду записей, каждая из которых может 
быть передана отдельно, что позволяет избежать фрагментации IP.

При передаче сообщения подтверждения отправитель делит сообщение на серию из N смежных диапазонов данных.
Эти диапазоны НЕ ДОЛЖНЫ быть больше максимального размера фрагмента квитирования и ДОЛЖНЫ вместе содержать 
все сообщение квитирования. Диапазоны НЕ ДОЛЖНЫ перекрываться. Затем отправитель создает N сообщений подтверждения,
все с тем же значением message_seq, что и исходное сообщение подтверждения. Каждое новое сообщение помечается
fragment_offset (количество байтов, содержащихся в предыдущих фрагментах) и fragment_length (длина этого фрагмента).
 Поле длины во всех сообщениях такое же, как поле длины исходного сообщения. Не фрагментированное сообщение 
 - это вырожденный случай с fragment_offset = 0 и fragment_length = length.

Когда реализация DTLS получает фрагмент сообщения подтверждения, она ДОЛЖНА буферизовать его до тех пор, пока 
не получит все сообщение подтверждения. Реализации DTLS ДОЛЖНЫ быть способны обрабатывать перекрывающиеся диапазоны
фрагментов. Это позволяет отправителям повторно передавать сообщения подтверждения с меньшими размерами фрагментов,
если оценка PMTU изменяется. Обратите внимание, что, как и в случае с TLS, несколько сообщений рукопожатия могут быть 
помещены в одну и ту же запись DTLS при условии, что есть место и что они являются частью одной и той же проверки. 
Таким образом, есть два приемлемых способа упаковать два сообщения DTLS в одну дейтаграмму: в одну и ту же запись
 или в отдельные записи.

### 4.2.4. Тайм-аут и повторная передача

Сообщения DTLS сгруппированы в серию пакетов сообщений в соответствии с приведенными ниже диаграммами. Хотя каждая
серия сообщений может состоять из нескольких сообщений, их следует рассматривать как монолитные с точки зрения
тайм-аута и повторной передачи.

       Client                                          Server
       ------                                          ------
    
       ClientHello             -------->                           Flight 1
    
                               <-------    HelloVerifyRequest      Flight 2
    
       ClientHello             -------->                           Flight 3
    
                                                  ServerHello    \
                                                 Certificate*     \
                                           ServerKeyExchange*      Flight 4
                                          CertificateRequest*     /
                               <--------      ServerHelloDone    /
    
       Certificate*                                              \
       ClientKeyExchange                                          \
       CertificateVerify*                                          Flight 5
       [ChangeCipherSpec]                                         /
       Finished                -------->                         /
    
                                           [ChangeCipherSpec]    \ Flight 6
                               <--------             Finished    /
    
                   Figure 1. Message Flights for Full Handshake


​    
​       Client                                           Server
​       ------                                           ------
​    
       ClientHello             -------->                          Flight 1
    
                                                  ServerHello    \
                                           [ChangeCipherSpec]     Flight 2
                                <--------             Finished    /
    
       [ChangeCipherSpec]                                         \Flight 3
       Finished                 -------->                         /
    
         Figure 2. Message Flights for Session-Resuming Handshake
                           (No Cookie Exchange)

DTLS использует простую схему тайм-аута и повторной передачи со следующим конечным автоматом. Поскольку клиенты DTLS
отправляют первое сообщение (ClientHello), они запускаются в состоянии ПОДГОТОВКА. Серверы DTLS запускаются в состоянии
WAITING, но с пустыми буферами и без таймера повторной передачи.



                      +-----------+
                      | PREPARING |
                +---> |           | <--------------------+
                |     |           |                      |
                |     +-----------+                      |
                |           |                            |
                |           | Buffer next flight         |
                |           |                            |
                |          \|/                           |
                |     +-----------+                      |
                |     |           |                      |
                |     |  SENDING  |<------------------+  |
                |     |           |                   |  | Send
                |     +-----------+                   |  | HelloRequest
        Receive |           |                         |  |
           next |           | Send flight             |  | or
         flight |  +--------+                         |  |
                |  |        | Set retransmit timer    |  | Receive
                |  |       \|/                        |  | HelloRequest
                |  |  +-----------+                   |  | Send
                |  |  |           |                   |  | ClientHello
                +--)--|  WAITING  |-------------------+  |
                |  |  |           |   Timer expires   |  |
                |  |  +-----------+                   |  |
                |  |         |                        |  |
                |  |         |                        |  |
                |  |         +------------------------+  |
                |  |                Read retransmit      |
        Receive |  |                                     |
           last |  |                                     |
         flight |  |                                     |
                |  |                                     |
               \|/\|/                                    |
                                                         |
            +-----------+                                |
            |           |                                |
            | FINISHED  | -------------------------------+
            |           |
            +-----------+
                 |  /|\
                 |   |
                 |   |
                 +---+
    
              Read retransmit
           Retransmit last flight
    
          Figure 3. DTLS Timeout and Retransmission State Machine


Конечный автомат имеет три основных состояния.

В состоянии ПОДГОТОВКА реализация выполняет все вычисления, необходимые для подготовки следующей серии сообщений. 
Затем он буферизует их для передачи (сначала очищая буфер) и переходит в состояние SENDING.

В состоянии SENDING реализация передает буферизованную серию сообщений. После отправки сообщений реализация переходит
в состояние FINISHED, если это последний рейс в рукопожатии. Или, если реализация ожидает получить больше сообщений,
она устанавливает таймер повторной передачи и затем переходит в состояние WAITING.

Есть три способа выйти из состояния WAITING:

1. Истекает таймер повторной передачи: реализация переходит в состояние SENDING, где повторно передает полет, 
сбрасывает таймер повторной передачи и возвращается в состояние WAITING.

2. Реализация считывает повторно переданный полет от партнера: реализация переходит в состояние SENDING, где повторно 
передает полет, сбрасывает таймер повторной передачи и возвращается в состояние WAITING. Обоснованием здесь является 
то, что получение дублированного сообщения является вероятным результатом истечения таймера на одноранговом узле и,
 следовательно, предполагает, что часть предыдущего полета была потеряна.

3. Реализация получает следующую серию сообщений: если это последняя серия сообщений, реализация переходит к FINISHED.
 Если реализации необходимо отправить новый полет, она переходит в состояние ПОДГОТОВКА. Частичные чтения (будь то 
    частичные сообщения или только некоторые из сообщений в полете) не вызывают смены состояний или сброса таймера.

Поскольку клиенты DTLS отправляют первое сообщение (ClientHello), они запускаются в состоянии ПОДГОТОВКА. Серверы DTLS
 запускаются в состоянии WAITING, но с пустыми буферами и без таймера повторной передачи.

Когда серверу требуется повторное встряхивание, он переходит из состояния FINISHED в состояние PREPARING для передачи
 HelloRequest. Когда клиент получает HelloRequest, он переходит из FINISHED в PREPARING для передачи ClientHello.

Кроме того, как минимум в два раза превышающий MSL по умолчанию, определенный для [TCP], в состоянии FINISHED узел,
который передает последний полет (сервер в обычном рукопожатии или клиент в возобновленном рукопожатии) ДОЛЖЕН
ответить на повторную передачу последний полет партнера с ретрансляцией последнего полета. Это позволяет избежать 
тупиковых ситуаций в случае потери последнего рейса. Это требование применимо и к DTLS 1.0, и хотя в [DTLS1] не
указано явно, оно всегда требовалось для правильной работы конечного автомата. Чтобы понять, почему это необходимо,
рассмотрим, что происходит при обычном рукопожатии, если серверное сообщение Finished потеряно: сервер считает, 
что рукопожатие завершено, но на самом деле это не так. Поскольку клиент ожидает сообщения Finished, таймер
повторной передачи клиента срабатывает, и он повторно передает сообщение Finished клиента. Это заставит сервер 
ответить своим собственным сообщением Finished, завершив рукопожатие. Та же логика применяется на стороне
сервера для возобновленного рукопожатия.

Обратите внимание, что из-за потери пакетов одна сторона может отправлять данные приложения, даже если другая сторона
не получила сообщение Finished первой стороны. Реализации ДОЛЖНЫ либо отбрасывать, либо буферизовать все пакеты данных
приложения для новой эпохи, пока они не получат сообщение Finished для этой эпохи. Реализации МОГУТ рассматривать 
получение данных приложения с новой эпохой до получения соответствующего сообщения Finished как свидетельство 
переупорядочения или потери пакетов и немедленно повторно передавать свой последний полет, сокращая таймер повторной 
передачи.  

#### 4.2.4.1. Значения таймера

Хотя значения таймера являются выбором реализации, неправильное обращение с таймером может привести к серьезным 
проблемам с перегрузкой; например, если время ожидания многих экземпляров DTLS преждевременно истекает и повторная 
передача выполняется слишком быстро по перегруженному каналу. Реализации ДОЛЖНЫ использовать начальное значение 
таймера в 1 секунду (минимум, определенный в RFC 6298 [RFC6298]) и удваивать значение при каждой повторной передаче, 
но не менее, чем максимум 60 секунд, указанный в RFC 6298. Обратите внимание, что мы рекомендуем 1-секундный таймер, 
а не 3-секундный таймер по умолчанию RFC 6298, чтобы уменьшить задержку для приложений, чувствительных ко времени. 
Поскольку DTLS использует только повторную передачу для подтверждения, а не поток данных, влияние на перегрузку 
должно быть минимальным.

Реализации ДОЛЖНЫ сохранять текущее значение таймера до тех пор, пока не произойдет передача без потерь, после чего 
значение может быть сброшено до начального значения. После длительного периода бездействия, не менее чем в 10 раз 
превышающего текущее значение таймера, реализации могут сбросить таймер на начальное значение. Одна из ситуаций, когда 
это может произойти, - это когда повторное встряхивание используется после существенной передачи данных.


### 4.2.5. ChangeCipherSpec

Как и в случае с TLS, сообщение ChangeCipherSpec технически не является сообщением подтверждения, но ДОЛЖНО 
рассматриваться как часть того же полета, что и связанное сообщение Finished, для целей тайм-аута и повторной передачи. 
Это создает потенциальную двусмысленность, поскольку порядок ChangeCipherSpec не может быть однозначно установлен 
в отношении сообщений подтверждения в случае потери сообщения.

Это не проблема с любым текущим режимом TLS, потому что ожидаемый набор сообщений подтверждения, логически 
предшествующий ChangeCipherSpec, предсказуем по остальной части состояния подтверждения. Однако в будущих режимах 
НЕОБХОДИМО избегать неоднозначности.

### 4.2.6. CertificateVerify и завершенные сообщения

Сообщения CertificateVerify и Finished имеют тот же формат, что и в TLS. Вычисления хэша включают в себя все сообщения 
рукопожатия, включая специфичные для DTLS поля: message_seq, fragment_offset и fragment_length. Однако для устранения 
чувствительности к фрагментации сообщения подтверждения окончательный MAC ДОЛЖЕН вычисляться так, как если бы каждое 
сообщение подтверждения было отправлено как отдельный фрагмент. Обратите внимание, что в случаях, когда используется 
обмен файлами cookie, начальные ClientHello и HelloVerifyRequest НЕ ДОЛЖНЫ включаться в вычисления CertificateVerify 
или Finished MAC.

### 4.2.7. Предупреждающие сообщения

Обратите внимание, что предупреждающие сообщения вообще не передаются повторно, даже если они происходят в контексте 
рукопожатия. Однако реализации DTLS, которая обычно выдает предупреждение, СЛЕДУЕТ генерировать новое сообщение с 
предупреждением, если запись о нарушении получена снова (например, в виде повторно переданного сообщения подтверждения). 
Реализации ДОЛЖНЫ обнаруживать, когда одноранговый узел постоянно отправляет плохие сообщения, и завершать состояние 
локального соединения после обнаружения такого неправильного поведения.

### 4.2.8. Установление новых ассоциаций с существующими параметрами

Если пара клиент-сервер DTLS настроена таким образом, что повторяющиеся подключения происходят на одном и том же 
квартете хост / порт, то возможно, что клиент молча откажется от одного подключения, а затем инициирует другое с теми 
же параметрами (например, после перезагружать). Это будет отображаться на сервере как новое рукопожатие с эпохой = 0. 
В случаях, когда сервер считает, что у него есть существующая ассоциация на данном квартете хоста / порта и он 
получает epoch = 0 ClientHello, ему СЛЕДУЕТ продолжить новое рукопожатие, но НЕ ДОЛЖНО уничтожать существующую 
ассоциацию, пока клиент не продемонстрирует достижимость, либо завершив обмен файлами cookie или завершение 
полного рукопожатия, включая доставку проверяемого сообщения Finished. После получения правильного сообщения Finished 
сервер ДОЛЖЕН отказаться от предыдущей ассоциации, чтобы избежать путаницы между двумя действительными ассоциациями с 
перекрывающимися эпохами. Требование доступности не позволяет злоумышленникам, находящимся вне пути / слепым, 
разрушать ассоциации просто путем отправки поддельного ClientHellos.

## 4.3. Резюме нового синтаксиса

Этот раздел включает спецификации структур данных, которые изменились между TLS 1.2 и DTLS 1.2.
См. [TLS12] для определения этого синтаксиса.

### 4.3.1.  Record Layer

      struct {
        ContentType type;
        ProtocolVersion version;
        uint16 epoch;                                     // New field
        uint48 sequence_number;                           // New field
        uint16 length;
        opaque fragment[DTLSPlaintext.length];
      } DTLSPlaintext;
    
      struct {
        ContentType type;
        ProtocolVersion version;
        uint16 epoch;                                     // New field
        uint48 sequence_number;                           // New field
        uint16 length;
        opaque fragment[DTLSCompressed.length];
      } DTLSCompressed;
    
      struct {
        ContentType type;
        ProtocolVersion version;
        uint16 epoch;                                     // New field
        uint48 sequence_number;                           // New field
        uint16 length;
        select (CipherSpec.cipher_type) {
          case block:  GenericBlockCipher;
          case aead:   GenericAEADCipher;                 // New field
        } fragment;
      } DTLSCiphertext;

### 4.3.2.  Handshake Protocol

       enum {
         hello_request(0), client_hello(1), server_hello(2),
         hello_verify_request(3),                          // New field
         certificate(11), server_key_exchange (12),
         certificate_request(13), server_hello_done(14),
         certificate_verify(15), client_key_exchange(16),
         finished(20), (255) } HandshakeType;
    
       struct {
         HandshakeType msg_type;
         uint24 length;
         uint16 message_seq;                               // New field
         uint24 fragment_offset;                           // New field
         uint24 fragment_length;                           // New field
         select (HandshakeType) {
           case hello_request: HelloRequest;
           case client_hello:  ClientHello;
           case server_hello:  ServerHello;
           case hello_verify_request: HelloVerifyRequest;  // New field
           case certificate:Certificate;
           case server_key_exchange: ServerKeyExchange;
           case certificate_request: CertificateRequest;
           case server_hello_done:ServerHelloDone;
           case certificate_verify:  CertificateVerify;
           case client_key_exchange: ClientKeyExchange;
           case finished: Finished;
         } body; } Handshake;
    
       struct {
         ProtocolVersion client_version;
         Random random;
         SessionID session_id;
         opaque cookie<0..2^8-1>;                             // New field
         CipherSuite cipher_suites<2..2^16-1>;
         CompressionMethod compression_methods<1..2^8-1>; } ClientHello;
    
       struct {
         ProtocolVersion server_version;
         opaque cookie<0..2^8-1>; } HelloVerifyRequest;

# 5. Соображения безопасности

   This document describes a variant of TLS 1.2; therefore, most of the
   security considerations are the same as those of TLS 1.2 [TLS12],
   described in Appendices D, E, and F.

   The primary additional security consideration raised by DTLS is that
   of denial of service.  DTLS includes a cookie exchange designed to
   protect against denial of service.  However, implementations that do
   not use this cookie exchange are still vulnerable to DoS.  In
   particular, DTLS servers that do not use the cookie exchange may be
   used as attack amplifiers even if they themselves are not
   experiencing DoS.  Therefore, DTLS servers SHOULD use the cookie
   exchange unless there is good reason to believe that amplification is
   not a threat in their environment.  Clients MUST be prepared to do a
   cookie exchange with every handshake.

   Unlike TLS implementations, DTLS implementations SHOULD NOT respond
   to invalid records by terminating the connection.  See Section
   4.1.2.7 for details on this.

# 6. Благодарности

   The authors would like to thank Dan Boneh, Eu-Jin Goh, Russ Housley,
   Constantine Sapuntzakis, and Hovav Shacham for discussions and
   comments on the design of DTLS.  Thanks to the anonymous NDSS
   reviewers of our original NDSS paper on DTLS [DTLS] for their
   comments.  Also, thanks to Steve Kent for feedback that helped
   clarify many points.  The section on PMTU was cribbed from the DCCP
   specification [DCCP].  Pasi Eronen provided a detailed review of this
   specification.  Peter Saint-Andre provided the list of changes in
   Section 8.  Helpful comments on the document were also received from
   Mark Allman, Jari Arkko, Mohamed Badra, Michael D'Errico, Adrian
   Farrell, Joel Halpern, Ted Hardie, Charlia Kaufman, Pekka Savola,
   Allison Mankin, Nikos Mavrogiannopoulos, Alexey Melnikov, Robin
   Seggelmann, Michael Tuexen, Juho Vaha-Herttua, and Florian Weimer.

# 7.  IANA Considerations

   This document uses the same identifier space as TLS [TLS12], so no
   new IANA registries are required.  When new identifiers are assigned
   for TLS, authors MUST specify whether they are suitable for DTLS.
   IANA has modified all TLS parameter registries to add a DTLS-OK flag,
   indicating whether the specification may be used with DTLS.  At the
   time of publication, all of the [TLS12] registrations except the
   following are suitable for DTLS.  The full table of registrations is
   available at [IANA].

   From the TLS Cipher Suite Registry:

      0x00,0x03 TLS_RSA_EXPORT_WITH_RC4_40_MD5        [RFC4346]
      0x00,0x04 TLS_RSA_WITH_RC4_128_MD5              [RFC5246]
      0x00,0x05 TLS_RSA_WITH_RC4_128_SHA              [RFC5246]
      0x00,0x17 TLS_DH_anon_EXPORT_WITH_RC4_40_MD5    [RFC4346]
      0x00,0x18 TLS_DH_anon_WITH_RC4_128_MD5          [RFC5246]
      0x00,0x20 TLS_KRB5_WITH_RC4_128_SHA             [RFC2712]
      0x00,0x24 TLS_KRB5_WITH_RC4_128_MD5             [RFC2712]
      0x00,0x28 TLS_KRB5_EXPORT_WITH_RC4_40_SHA       [RFC2712]
      0x00,0x2B TLS_KRB5_EXPORT_WITH_RC4_40_MD5       [RFC2712]
      0x00,0x8A TLS_PSK_WITH_RC4_128_SHA              [RFC4279]
      0x00,0x8E TLS_DHE_PSK_WITH_RC4_128_SHA          [RFC4279]
      0x00,0x92 TLS_RSA_PSK_WITH_RC4_128_SHA          [RFC4279]
      0xC0,0x02 TLS_ECDH_ECDSA_WITH_RC4_128_SHA       [RFC4492]
      0xC0,0x07 TLS_ECDHE_ECDSA_WITH_RC4_128_SHA      [RFC4492]
      0xC0,0x0C TLS_ECDH_RSA_WITH_RC4_128_SHA         [RFC4492]
      0xC0,0x11 TLS_ECDHE_RSA_WITH_RC4_128_SHA        [RFC4492]
      0xC0,0x16 TLS_ECDH_anon_WITH_RC4_128_SHA        [RFC4492]
      0xC0,0x33 TLS_ECDHE_PSK_WITH_RC4_128_SHA        [RFC5489]

   From the TLS Exporter Label Registry:

      client EAP encryption       [RFC5216]
      ttls   keying material      [RFC5281]
      ttls   challenge            [RFC5281]

   This document defines a new handshake message, hello_verify_request,
   whose value has been allocated from the TLS HandshakeType registry
   defined in [TLS12].  The value "3" has been assigned by the IANA.

# 8.  Changes since DTLS 1.0

   This document reflects the following changes since DTLS 1.0 [DTLS1].

   -  Updated to match TLS 1.2 [TLS12].

   -  Addition of AEAD Ciphers in Section 4.1.2.3 (tracking changes in
      TLS 1.2.

   -  Clarifications regarding sequence numbers and epochs in Section
      4.1 and a clear procedure for dealing with state loss in Section
      4.2.8.

   -  Clarifications and more detailed rules regarding Path MTU issues
      in Section 4.1.1.1. Clarification of the fragmentation text
      throughout.

   -  Clarifications regarding handling of invalid records in Section
      4.1.2.7.

   -  A new paragraph describing handling of invalid cookies at the end
      of Section 4.2.1.

   -  Some new text describing how to avoid handshake deadlock
      conditions at the end of Section 4.2.4.

   -  Some new text about CertificateVerify messages in Section 4.2.6.

   -  A prohibition on epoch wrapping in Section 4.1.

   -  Clarification of the IANA requirements and the explicit
      requirement for a new IANA registration flag for each parameter.

   -  Added a record sequence number mirroring technique for handling
      repeated ClientHello messages.

   -  Recommend a fixed version number for HelloVerifyRequest.

   -  Numerous editorial changes.

# 9.  References

## 9.1.  Normative References

   [REQ]       Bradner, S., "Key words for use in RFCs to Indicate
               Requirement Levels", BCP 14, RFC 2119, March 1997.

   [RFC1191]   Mogul, J. and S. Deering, "Path MTU discovery", RFC 1191,
               November 1990.

   [RFC4301]   Kent, S. and K. Seo, "Security Architecture for the
               Internet Protocol", RFC 4301, December 2005.

   [RFC4443]   Conta, A., Deering, S., and M. Gupta, Ed., "Internet
               Control Message Protocol (ICMPv6) for the Internet
               Protocol Version 6 (IPv6) Specification", RFC 4443, March
               2006.

   [RFC4821]   Mathis, M. and J. Heffner, "Packetization Layer Path MTU
               Discovery", RFC 4821, March 2007.

   [RFC6298]   Paxson, V., Allman, M., Chu, J., and M. Sargent,
               "Computing TCP's Retransmission Timer", RFC 6298, June
               2011.

   [RSAGCM]    Salowey, J., Choudhury, A., and D. McGrew, "AES Galois
               Counter Mode (GCM) Cipher Suites for TLS", RFC 5288,
               August 2008.

   [TCP]       Postel, J., "Transmission Control Protocol", STD 7, RFC
               793, September 1981.

   [TLS12]     Dierks, T. and E. Rescorla, "The Transport Layer Security
               (TLS) Protocol Version 1.2", RFC 5246, August 2008.

## 9.2.  Informative References

   [DCCP]      Kohler, E., Handley, M., and S. Floyd, "Datagram
               Congestion Control Protocol (DCCP)", RFC 4340, March
               2006.

   [DCCPDTLS]  Phelan, T., "Datagram Transport Layer Security (DTLS)
               over the Datagram Congestion Control Protocol (DCCP)",
               RFC 5238, May 2008.

   [DTLS]      Modadugu, N. and E. Rescorla, "The Design and
               Implementation of Datagram TLS", Proceedings of ISOC NDSS
               2004, February 2004.

   [DTLS1]     Rescorla, E. and N. Modadugu, "Datagram Transport Layer
               Security", RFC 4347, April 2006.

   [ECCGCM]    Rescorla, E., "TLS Elliptic Curve Cipher Suites with
               SHA-256/384 and AES Galois Counter Mode (GCM)", RFC 5289,
               August 2008.

   [ESP]       Kent, S., "IP Encapsulating Security Payload (ESP)", RFC
               4303, December 2005.

   [IANA]      IANA, "Transport Layer Security (TLS) Parameters",
               http://www.iana.org/assignments/tls-parameters.

   [IKEv2]     Kaufman, C., Hoffman, P., Nir, Y., and P. Eronen,
               "Internet Key Exchange Protocol Version 2 (IKEv2)", RFC
               5996, September 2010.

   [IMAP]      Crispin, M., "INTERNET MESSAGE ACCESS PROTOCOL - VERSION
               4rev1", RFC 3501, March 2003.

   [PHOTURIS]  Karn, P. and W. Simpson, "Photuris: Session-Key
               Management Protocol", RFC 2522, March 1999.

   [POP]       Myers, J. and M. Rose, "Post Office Protocol - Version
               3", STD 53, RFC 1939, May 1996.

   [SIP]       Rosenberg, J., Schulzrinne, H., Camarillo, G., Johnston,
               A., Peterson, J., Sparks, R., Handley, M., and E.
               Schooler, "SIP: Session Initiation Protocol", RFC 3261,
               June 2002.

   [TLS]       Dierks, T. and C. Allen, "The TLS Protocol Version 1.0",
               RFC 2246, January 1999.

   [TLS11]     Dierks, T. and E. Rescorla, "The Transport Layer Security
               (TLS) Protocol Version 1.1", RFC 4346, April 2006.

   [WHYIPSEC]  Bellovin, S., "Guidelines for Specifying the Use of IPsec
               Version 2", BCP 146, RFC 5406, February 2009.

Authors' Addresses

   Eric Rescorla
   RTFM, Inc.
   2064 Edgewood Drive
   Palo Alto, CA 94303

   EMail: ekr@rtfm.com


   Nagendra Modadugu
   Google, Inc.

   EMail: nagendra@cs.stanford.edu




























Rescorla & Modadugu          Standards Track                   [Page 32]
