+++
date = "2014-11-01T17:50:10+01:00"
draft = false
title = "Proxy LDAP en Golang"
slug = "ldap-en-golang"

+++

Bonjour !

Comme promis dans mon article précédent je vais expliquer aujourd'hui la mise en application du langage Golang pour implementer le protocole LDAP.

# LDAP
           
LDAP est un protocole permettant d'interagir avec un annuaire (LDAP = Lightweight Directory Access Protocol). Il permet à un client d'effectuer toutes sorte d'opérations standards auprès d'un serveur d'annuaire: authentification, interrogation, mise à jour et suppression d'entrées...

De protocole, LDAP a ensuite évolué pour représenter une norme pour les systèmes d'annuaires, incluant un modèle de données, un modèle de nommage, un modèle fonctionnel basé sur le protocole LDAP, un modèle de sécurité et un modèle de réplication (source [Wikipedia](http://fr.wikipedia.org/wiki/Lightweight_Directory_Access_Protocol)).

LDAP est très répandu, son implémentation existe donc déjà dans de nombreux langages. Une implémentation très connue est *Microsoft Active Directory*. Pour tester mes développements j'ai utilisé [*Apache Directory Studio*](http://directory.apache.org/studio/), qui propose un client et un serveur LDAP, le tout embarqué dans une interface Eclipse.



## Messages

### Enveloppe

La structure des messages LDAP est clairement définie dans la [RFC 4511](http://www.ietf.org/rfc/rfc4511.txt) en utilisant la notation ASN.1 ([Abstract Syntax Notation One](http://en.wikipedia.org/wiki/Abstract_Syntax_Notation_One)). L'enveloppe de base de chaque message se compose ainsi:

	LDAPMessage ::= SEQUENCE {
             messageID       MessageID,
             protocolOp      CHOICE {
                  bindRequest           BindRequest,
                  bindResponse          BindResponse,
                  unbindRequest         UnbindRequest,
                  searchRequest         SearchRequest,
                  searchResEntry        SearchResultEntry,
                  searchResDone         SearchResultDone,
                  searchResRef          SearchResultReference,
                  modifyRequest         ModifyRequest,
                  modifyResponse        ModifyResponse,
                  addRequest            AddRequest,
                  addResponse           AddResponse,
                  delRequest            DelRequest,
                  delResponse           DelResponse,
                  modDNRequest          ModifyDNRequest,
                  modDNResponse         ModifyDNResponse,
                  compareRequest        CompareRequest,
                  compareResponse       CompareResponse,
                  abandonRequest        AbandonRequest,
                  extendedReq           ExtendedRequest,
                  extendedResp          ExtendedResponse,
                  ...,
                  intermediateResponse  IntermediateResponse },
             controls       [0] Controls OPTIONAL }
             

Un message est donc composé de plusieurs attributs:

* un identifiant
* une opération (connexion, recherche, modification...)
* des options de contrôles de l'opération

L'opération et les contrôles sont eux-même composés d'autres attributs. La RFC4511 fourni de manière similaire la définition de toutes les structures possibles pour chaque opération permise par le protocole: bind, unbind, search ...

Un message LDAP est donc une structure composée que l'on pourra facilement représenter par des *struct* en langage Golang.

### Sérialisation

Une fois construit, un message LDAP doit être sérialisé pour être transmis sur le réseau.  Il faut aussi que l'interlocuteur sache le désérialiser de la même manière.

La norme ASN.1, utilisée plus haut pour décrire les messages, spécifie plusieurs normes pour encoder un message en une suite d'octets. La RFC de LDAP (RFC4511) spécifie que l'encodage BER ([Basic Encoding Rule](http://en.wikipedia.org/wiki/X.690#BER_encoding)) doit être utilisé avec quelques restrictions:

> 5.1.  Protocol Encoding
    
>   The protocol elements of LDAP SHALL be encoded for exchange using the Basic Encoding Rules [BER] of [ASN.1] with the following restrictions:

> - Only the definite form of length encoding is used.
> - OCTET STRING values are encoded in the primitive form only.
> - If the value of a BOOLEAN type is true, the encoding of the value octet is set to hex "FF".
> - If a value of a type is its default value, it is absent.  Only some BOOLEAN and INTEGER types have default values in this protocol definition.

> These restrictions are meant to ease the overhead of encoding and decoding certain elements in BER.

> These restrictions do not apply to ASN.1 types encapsulated inside of OCTET STRING values, such as attribute values, unless otherwise stated.
    
Une structure encodée au format BER se présente sous une suite d'octets qui en définissent le type, la longueur et les données. Pour une structure composée les données sont elles-mêmes une ou plusieurs sous-structures au format BER.
    
![](/images/2014/Oct/asn1.png)

Maintenant que nous savons un peu plus où nous mettons les pieds, passons au développement.

# Développement

## Premier essai: encoding/asn1

Avant de réinventer la roue, je suis allé voir ce que proposait Golang. La librairie *encoding/asn1* fourni des fonctions permettant de lire et d'écrire des structures mais elle présente plusieurs inconvénients:

* l'encodage se fait au format DER au lieu de BER, car la librairie est utilisée surtout pour lire les certificats SSL et non pour des structures LDAP
* certaines fonctionnalités sont manquantes: les tags applicatifs ne sont pas supportés par la librairie d'encodage
* on doit connaître à l'avance le type de message que l'on est en train de décoder, en particulier la valeur du tag alors qu'elle est fonction du type de message LDAP. Dans l'exemple ci-dessous (issu de la librairie de test *encoding/asn1/asn1_test.go*), l'annotation en face du champ *A* illustre ce problème:

		type TestContextSpecificTags2 struct {
			A int `asn1:"explicit,tag:1"`
			B int
		}

Malgré ses manques, la librairie fourni certaines fonctions de bases bien utiles:

* lecture des tags
* lectures des valeurs de bases (entier, octet strings...)

C'est lectures se font au format DER, qui reste un encodage BER valable bien que plus restrictif. Ces fonctions constituent donc une bonne base que j'ai utilisée dans un premier temps mais que j'adapterai par la suite.

Ce premier contact avec l'encodage ASN.1 était instructif. Pour expérimenter, je recommande ce petit outil en javascript bien pratique: http://lapo.it/asn1js/. La page Wiki de [BER](http://en.wikipedia.org/wiki/X.690#BER_encoding) est également une bonne base.


## RFC 4511

La [RFC 4511](http://www.ietf.org/rfc/rfc4511.txt) a constitué mon point de départ. Elle détaille:

* la structure des messages LDAP
* la manière dont le client et le serveur doivent dialoguer (identifiants de messages uniques et incrémentaux...)
* les formats à respecter pour certains types de données spécifiques (*distinguished names*, *numeric oid*...)
* comment le serveur doit traiter chaque type de message (recherche, ajout, suppression, comparaison...)
* la liste de toutes les autres RFC nécessaires (par exemple la RFC4512 pour le format des *numeric oid*...) 
* les codes retours possibles et leurs significations

J'ai commencé par coder l'ensemble des structures des messages LDAP.


## Structures

La RFC comporte une annexe récapitulant l'ensemble des structures des messages LDAP. Pour être sûr de ne rien oublier, j'ai copié cette partie dans un nouveau fichier Golang, je l'ai commentée et j'ai commencé à coder au milieu de cette RFC.


Lorsque la RFC énonce ceci:

    //        Control ::= SEQUENCE {
    //             controlType             LDAPOID,
    //             criticality             BOOLEAN DEFAULT FALSE,
    //             controlValue            OCTET STRING OPTIONAL }
    
Je l'ai traduit par la structure Golang suivante:
    
    type Control struct {
        controlType  LDAPOID
        criticality  BOOLEAN
        controlValue *OCTETSTRING
    }


Cette structure utilise d'autres types également définis par la RFC, par exemple LDAPOID:

    //        LDAPOID ::= OCTET STRING -- Constrained to <numericoid>
    //                                 -- [RFC4512]
    type LDAPOID OCTETSTRING

On arrive au final aux types de base de Golang, par exemple un *int32* pour les types INTEGER:

    //        MessageID ::= INTEGER (0 ..  maxInt)
    //
    type MessageID INTEGER

    //        maxInt INTEGER ::= 2147483647 -- (2^^31 - 1) --
    const maxInt = INTEGER(2147483647)

    type INTEGER int32


J'ai également utilisé les constantes définies par la RFC, ainsi voici le type *BindRequest*. Vous remarquerez le tag applicatif *0* et les valeurs min et max de la version:

    //        BindRequest ::= [APPLICATION 0] SEQUENCE {
    //             version                 INTEGER (1 ..  127),
    //             name                    LDAPDN,
    //             authentication          AuthenticationChoice }
    const TagBindRequest = 0
    const BindRequestVersionMin = 1
    const BindRequestVersionMax = 127
    
    type BindRequest struct {
        version        INTEGER
        name           LDAPDN
        authentication AuthenticationChoice
    }



J'ai utilisé cette méthode de manière systématique pour traduire l'ensemble des structures des messages. Une fois terminé, le fichier *struct.go* est auto-documenté par la RFC et ne devrait plus varier, à moins que la RFC elle-même ne soit mise à jour.

Passons maintenant au décodage d'un flux d'octets en une structure LDAP Golang.

## Décodage des messages

*Remarque: Pour une meilleure lisibilité de l'article, les exemples de code sont simplifiés (j'ai enlevé la gestion des erreurs).*

J'ai repris la même méthode que pour les structures: copier la RFC dans un nouveau fichier Golang, la commenter et insérer les fonctions de décodage au milieu de cette RFC.

La composition des messages en structures imbriquées nous amène à un traitement récursif. Comme vu dans la partie *Sérialisation*, chaque flux d'octet à lire comporte un tag (type) et une longueur, suivis des données. On a donc un aspect systématique pour la lecture d'une structure:

*	lire le tag et vérifier que sa valeur correspond au contexte
*	lire la longueur
*	récupérer la quantité d'octets correspondant à la longueur
*	lire chaque attribut de la structure
*	si un attribut est une sous-structure, relancer la lecture sur le sous-flux d'octets correspondants  

Les trois premières étapes sont communes à toutes les structures. Je les ai factorisées dans une fonction *ReadSubBytes*:

    func (b Bytes) ReadSubBytes(class int, tag int, callback func(bytes Bytes)) {
        // Check tag
        tagAndLength := b.ParseTagAndLength()
        tagAndLength.Expect(class, tag, isCompound)
            
        // Create sub-bytes
        start := b.offset
        end := b.offset + tagAndLength.Length        
        subBytes := Bytes{offset: 0, bytes: b.bytes[start:end]}
        
        // Recursive process 
        callback(subBytes)
        
        return
    }

Les deux dernières étapes (lecture des attributs) dépendent du contexte. J'ai créé une fonction *readComponents* spécifique à chaque type. Ainsi pour la structure *BindRequest*:

    //        BindRequest ::= [APPLICATION 0] SEQUENCE {
    //             version                 INTEGER (1 ..  127),
    //             name                    LDAPDN,
    //             authentication          AuthenticationChoice }

    func (bindrequest *BindRequest) readComponents(bytes Bytes) {
        bindrequest.version = readINTEGER(bytes)
        bindrequest.name = readLDAPDN(bytes)
        bindrequest.authentication = readAuthenticationChoice(bytes)
    }
    
La lecture d'une *bindRequest* se fait alors très simplement en appelant la fonction *ReadSubBytes* et en lui passant la fonction *readComponents* en paramètre:


    func readBindRequest(bytes Bytes) (bindrequest BindRequest) {
        bytes.ReadSubBytes(
        	classApplication,
            TagBindRequest,
            bindrequest.readComponents)
        return
    }
    

Comme pour les structures, j'ai utilisé cette méthode de manière systématique pour écrire l'ensemble des fonctions de décodages. Une fois terminé, le fichier *read.go* fait environ 1600 lignes, avec un unique point d'entrée: la fonction *ReadLDAPMessage*:

    func ReadLDAPMessage(bytes Bytes) (message LDAPMessage) {
        bytes.ReadSubBytes(
        	classUniversal,
            tagSequence,
            message.readLDAPMessageComponents)
        return
    }
    
Nous disposons ainsi d'une librairie capable de lire un flux d'octets et de retourner la structure LDAP correspondante. C'est une étape importante !

L'étape logique suivante serait de se lancer dans l'écriture des fonctions d'encodage. Mais avant ça, j'ai voulu me motiver en testant la librarie déjà écrite.


## Proxy LDAP

Nous disposons d'une librairie capable de convertir un tableau d'octets en un message LDAP. Nous sommes ainsi en mesure d'écouter le trafic LDAP entre un client et un serveur. Nous pouvons donc créer un proxy LDAP.

### Architecture

L'algorithme du proxy est simple:

* attendre une connection d'un client
* segmenter le flux réseaux du client en tableaux d'octets correspondant chacun à un seul message LDAP
* décoder la structure du message LDAP et l'afficher dans la console
* transmettre le tableau d'octets original au serveur
* procèder de manière symétrique pour afficher les réponses en provenance du serveur

Les goroutines nous permettent séparer au mieux les fonctions (découpage, décodage, transmission). Voici l'architecture que j'ai adoptée pour ce proxy LDAP:

![](/images/2014/Nov/ldap_proxy.jpg)

### Implémentation

Notre proxy comporte deux connexions et trois channels:

    type Proxy struct {
        name       string
        clientConn net.Conn
        serverConn net.Conn
        dumpChan   chan Message
        clientChan chan Message
        serverChan chan Message
    }

La boucle principale se contente d'ouvrir les connexions réseau et de démarrer le proxy:

    func Forward(local string, remote string) {
        listener := net.Listen("tcp", local)
        i := 0
        for {
            i++
            proxy := Proxy{name: fmt.Sprintf("PROXY%d", i)}
            proxy.clientConn = listener.Accept()
            proxy.serverConn = net.Dial("tcp", remote)
            go proxy.start()
        }
    }

Le démarrage du proxy crée les channels et lance les goroutines:

    func (p *Proxy) start() {
        p.dumpChan = make(chan Message)
        p.clientChan = make(chan Message)
        p.serverChan = make(chan Message)
        go p.dump()
        go p.readClient()
        go p.writeServer()
        go p.readServer()
        go p.writeClient()
    }

### Go routines

Les goroutines *readClient* et *readServer* sont similaires:

    func (p *Proxy) readClient() {
        for {
            bytes = p.readLdapMessageBytes(p.clientConn)
            message := Message{bytes: *bytes}
            p.dumpChan <- message
            p.serverChan <- message
        }
    }
    
Les goroutines *writeServer* et *writeClient* font juste du passe-plat pour le moment:
    
    func (p *Proxy) writeServer() {
        for msg := range p.serverChan {
            p.serverConn.Write(msg.bytes)
        }
    }
    
Enfin la goroutine de *dump*, qui utilise une librairie de formatage histoire de rendre les structures golang lisibles dans la console:
    
    func (p *Proxy) dump() {
        for msg := range p.dumpChan {
            // Now decode the message
            message := p.decodeMessage(msg.bytes)
            result = fmt.Sprintf("%# v", pretty.Formatter(message))
            log.Printf("Message: %s - %s - msg %d %s\n\n", p.name, msg.source, msg.id, result)
        }
    }  


### Test

Il suffit de lancer:

 * le serveur LDAP avec Apache Directory Studio
 * le proxy LDAP en ligne de commande
 * le client LDAP en le connectant au port d'écoute du proxy

La suite parle d'elle-même :-)

	$ go run proxy.go
    2014/10/25 17:53:52 Listening on port :2389...
    2014/10/25 17:54:07 New connection accepted
    2014/10/25 17:54:07 Message: PROXY1 - CLIENT - msg 2 
    message.LDAPMessage{
        messageID:  1,
        protocolOp: message.BindRequest{
            version:        3,
            name:           "",
            authentication: "",
        },
        controls: (*message.Controls)(nil),
    }
    
    2014/10/25 17:54:07 Message: PROXY1 - SERVER - msg 1 
    message.LDAPMessage{
        messageID:  1,
        protocolOp: message.BindResponse{},
        controls:   (*message.Controls)(nil),
    }
    
    2014/10/25 17:54:07 Message: PROXY1 - CLIENT - msg 3 
    message.LDAPMessage{
        messageID:  2,
        protocolOp: message.SearchRequest{
            baseObject:   "",
            scope:        0,
            derefAliases: 3,
            sizeLimit:    0,
            timeLimit:    0,
            typesOnly:    false,
            filter:       "objectClass",
            attributes:   {"subschemaSubentry"},
        },
        controls: (*message.Controls)(nil),
    }
    
    2014/10/25 17:54:07 Message: PROXY1 - SERVER - msg 2 
    message.LDAPMessage{
        messageID:  2,
        protocolOp: message.SearchResultEntry{
            objectName: "",
            attributes: {
                {
                    type_: "subschemaSubentry",
                    vals:  {"cn=schema"},
                },
            },
        },
        controls: (*message.Controls)(nil),
    }
    
    2014/10/25 17:54:07 Message: PROXY1 - SERVER - msg 3 
    message.LDAPMessage{
        messageID:  2,
        protocolOp: message.SearchResultDone{},
        controls:   (*message.Controls)(nil),
    }
    
    2014/10/25 17:54:07 Message: PROXY1 - CLIENT - msg 4 
    message.LDAPMessage{
        messageID:  3,
        protocolOp: message.SearchRequest{
            baseObject:   "cn=schema",
            scope:        0,
            derefAliases: 3,
            sizeLimit:    0,
            timeLimit:    0,
            typesOnly:    false,
            filter:       message.FilterEqualityMatch{attributeDesc:"objectClass", assertionValue:"subschema"},
            attributes:   {"createTimestamp", "modifyTimestamp"},
        },
        controls: (*message.Controls)(nil),
    }
    
    2014/10/25 17:54:07 Message: PROXY1 - SERVER - msg 4 
    message.LDAPMessage{
        messageID:  3,
        protocolOp: message.SearchResultEntry{
            objectName: "cn=schema",
            attributes: {
                {
                    type_: "modifyTimestamp",
                    vals:  {"20090818022733Z"},
                },
                {
                    type_: "createTimestamp",
                    vals:  {"20090818022733Z"},
                },
            },
        },
        controls: (*message.Controls)(nil),
    }
    
    2014/10/25 17:54:07 Message: PROXY1 - SERVER - msg 5 
    message.LDAPMessage{
        messageID:  3,
        protocolOp: message.SearchResultDone{},
        controls:   (*message.Controls)(nil),
    }
    
    2014/10/25 17:54:07 Message: PROXY1 - CLIENT - msg 5 
    message.LDAPMessage{
        messageID:  4,
        protocolOp: message.SearchRequest{
            baseObject:   "",
            scope:        0,
            derefAliases: 0,
            sizeLimit:    0,
            timeLimit:    0,
            typesOnly:    false,
            filter:       "objectClass",
            attributes:   {"namingContexts", "subschemaSubentry", "supportedLDAPVersion", "supportedSASLMechanisms", "supportedExtension", "supportedControl", "supportedFeatures", "vendorName", "vendorVersion", "+", "objectClass"},
        },
        controls: (*message.Controls)(nil),
    }
    
    2014/10/25 17:54:07 Message: PROXY1 - SERVER - msg 6 
    message.LDAPMessage{
        messageID:  4,
        protocolOp: message.SearchResultEntry{
            objectName: "",
            attributes: {
                {
                    type_: "vendorName",
                    vals:  {"Apache Software Foundation"},
                },
                {
                    type_: "vendorVersion",
                    vals:  {"2.0.0-M14"},
                },
                {
                    type_: "objectClass",
                    vals:  {"top", "extensibleObject"},
                },
                {
                    type_: "subschemaSubentry",
                    vals:  {"cn=schema"},
                },
                {
                    type_: "supportedLDAPVersion",
                    vals:  {"3"},
                },
                {
                    type_: "supportedControl",
                    vals:  {"2.16.840.1.113730.3.4.3", "1.3.6.1.4.1.4203.1.10.1", "2.16.840.1.113730.3.4.2", "1.3.6.1.4.1.4203.1.9.1.4", "1.3.6.1.4.1.42.2.27.8.5.1", "1.3.6.1.4.1.4203.1.9.1.1", "1.3.6.1.4.1.4203.1.9.1.3", "1.3.6.1.4.1.4203.1.9.1.2", "1.3.6.1.4.1.18060.0.0.1", "2.16.840.1.113730.3.4.7", "1.2.840.113556.1.4.319"},
                },
                {
                    type_: "supportedExtension",
                    vals:  {"1.3.6.1.4.1.1466.20036", "1.3.6.1.4.1.1466.20037", "1.3.6.1.4.1.18060.0.1.5", "1.3.6.1.4.1.18060.0.1.3", "1.3.6.1.4.1.4203.1.11.1"},
                },
                {
                    type_: "supportedSASLMechanisms",
                    vals:  {"NTLM", "GSSAPI", "GSS-SPNEGO", "CRAM-MD5", "SIMPLE", "DIGEST-MD5"},
                },
                {
                    type_: "entryUUID",
                    vals:  {"f290425c-8272-4e62-8a67-92b06f38dbf5"},
                },
                {
                    type_: "namingContexts",
                    vals:  {"ou=system", "ou=schema", "dc=example,dc=com", "ou=config"},
                },
                {
                    type_: "supportedFeatures",
                    vals:  {"1.3.6.1.4.1.4203.1.5.1"},
                },
            },
        },
        controls: (*message.Controls)(nil),
    }
    
    .....

# Evolutions

Ce proxy LDAP est un module que l'on peut brancher en frontal sur n'importe quel serveur LDAP. Il peut être modifié facilement pour répondre à certains besoins:

*	répartition de charge
*	contrôle d'accès
*	contrôle et modification de requêtes à la volée, voire suppression de requêtes (firewall LDAP ?)
*	diagnostique et qualité de service: vérification du temps de réponse du serveur, collecte de statistiques (par client, par type de requête...)

Evidemment, cela suppose que le trafic soit en clair. Les connections chiffrées (TLS) ne sont pas supportées pour le moment.

# Conclusion


Le but de tout ceci était surtout de m'améliorer en Golang plus que de faire du LDAP. J'ai mis plus de temps à me trouver un style de programmation qu'à implémenter la RFC. Une fois cette étape franchie, j'ai déroulé la RFC et le proxy a fonctionné presque du premier coup à ma grande surprise ! 

Une première version du code est disponible sur Github: https://github.com/lor00x/goldap. Cette version est sans aucune garantie et nécessite encore beaucoup de travail.

La prochaine étape sera l'encodage des structures LDAP.

A suivre !


