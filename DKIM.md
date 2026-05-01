# DKIM
#### Domain Keys Identified Mail

Permet de signer les emails afin de valider le domaine de l'expediteur.

Cle Privée : Signe le mail
Clef Publique : visilbe dans le DNS du domaine pour valider la signature

---

### Trouver la clef publique

*dig linkedin.com TXT* ne contient pas d'entrée DKIM....

On doit trouver le selecteur dans la signature de l'email : <img width="1499" height="71" alt="image" src="https://github.com/user-attachments/assets/f703b66c-3072-44c4-9a8a-c2601b453872" />

Ici s=d2048-202308-00 , ce qui nous permet de récupérer via dig la clef publique : <img width="1906" height="55" alt="image" src="https://github.com/user-attachments/assets/924a4146-0bc0-4b8e-af55-cdd4c4462de7" />

On peut donc définir la clef publique dans un fichier PEM : <img width="610" height="188" alt="image" src="https://github.com/user-attachments/assets/facda12c-e366-4fe9-ae0c-c8ca73a15e7f" />

Et la signature visible dans le mail dans un autre fichier après l'avoir décodée en base64 : <img width="940" height="79" alt="image" src="https://github.com/user-attachments/assets/7ef1a6b1-014b-49c4-b1f3-03006d0699f9" />

On peut donc déchiffrer la signature à l'aide le la clef, ce qui devrait nous donner le hash du mail : <img width="942" height="80" alt="image" src="https://github.com/user-attachments/assets/f125d43c-a62f-4428-8371-c996a8c2eb79" />

Il faut maintenant vérifer que ce hash correspond bien à celui de l'email, on va donc hasher les champs listés dans la signature présente dans l'email : *Date:From:Subject:MIME-Version:Content-Type:To:X-LinkedIn-Class: X-LinkedIn-Template:X-LinkedIn-fbl;*

Le résultat nous donne le hash suivant : **da361b5f67debb75f66f45415c7f65557fb14ec63205830d8f88e7a4e66b33d3** qui correpsond à la fin du hash déchiffré précedemment.

Cela est normal car la première partie viens décrire l'algorithme et les paramètres de hash utilisés. (ASN.1 - https://datatracker.ietf.org/doc/html/rfc8017#section-10 - https://lapo.it/asn1js/)
