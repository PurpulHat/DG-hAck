# Description du Challenge
Voici le tout dernier jeu de type Clicker à la mode !
Il y a même un système de trophées mais l'un d'eux a l'air particulièrement difficile à obtenir...
Et l'obtenir semble débloquer une récompense secrète !
Mais est-ce "humainement" possible ?

# Objectifs
Atteindre le niveau 20 afin de pouvoir débloquer un succès : ce succès nous donnera le flag dont nous avons besoin pour réussir ce CTF. Cependant, il est impossible pour un humain de jouer assez rapidement pour atteindre le niveau 20 avant de mourir à cause des monstres.
# Recon
Ce jeu vidéo, spécialement conçu pour la DG'HACK, est plutôt simple : il suffit de cliquer sur les fruits et les pièces pour monter en niveau.
![image](https://github.com/user-attachments/assets/1d868589-b304-4752-b833-1e815257cc07)
Cependant, il faut une vitesse surhumaine pour cliquer sur tous les objets avant de se faire tuer par des monstres, tels que les fantômes. De fait, à chaque partie, nous mourrons facilement au niveau ~8 avant de nous faire tuer et de devoir recommencer la partie.

Avec BurpSuite, on peut facilement remarquer que des requêtes POST se redirigent vers l'API du jeu. En regardant de plus près ces requêtes, on peut constater qu'elles ont pour but d'informer le serveur sur quoi nous avons cliqué dans le jeu. Par exemple, si nous avons cliqué sur une pièce, une requête POST avec un corps en JSON signifiera au jeu que nous avons cliqué dessus.

Ensuite, des requêtes GET nous seront envoyées pour nous prévenir de l'avancée de notre partie.

# Payload

Avant de commencer à construire le payload, j'ai fait une énumération de variables contenant à la fois les informations de base (URL / Cookie / Contenu du Shop / ...) et des listes contenant les monstres à tuer ou les objets à ne pas cliquer (Boo / bombe / ...).
```python
start = time.time()
url = "https://feedthisdragon3.chall.malicecyber.com/api/v1"
end_uuid = random.randrange(999999)
cookie = "uuid=cf86a4b9-2666-454d-b65c-b2970b811112; expires=Date; path=/\";"
bad_things = ['bomb', 'fox', 'life']
boo_list = ['null','lilboo', 'midboo', 'bigboo']
shop = []
number_buy = 0
a = 0 
b = 0
roulette = 0
door_list = [True, False, False, False, False, False, False, False, False, False, False, False, False, False, False, False, False]
error = 0
test = []
```

Avec ces informations, nous pouvons facilement créer un script disant "Clique sur tout ce qui n'est PAS mauvais."
```python
def RequestTheDragon(method, item=0, uuid=end_uuid, shop=0):
    global error
    ftd_header={
    "Cookie": f"uuid=df2572c5-361c-42de-9434-af9a9b{uuid};",
    "Authorization": "mynotsosecrettoken",
    "Session": f"df2572c5-361c-42de-9434-af9a9b{uuid}",
    "Itemuuid": f"{item}",
    "Shopuuid": f"{shop}",
    "Origin": "https://feedthisdragon3.chall.malicecyber.com",
    }

    if method == "get":
        request_the_dragon = requests.get(url, headers=ftd_header)
        if request_the_dragon.status_code != 200:
            print("ERROR GET Method :",request_the_dragon.status_code)
            RequestTheDragon("get")
            error =+ 1

    if method == "post":
        request_the_dragon = requests.post(url, headers=ftd_header)
        if request_the_dragon.status_code != 200:
            print("ERROR POST Method :",request_the_dragon.status_code)
            error =+ 1

[...]

while True:

[...]
 
    return json.loads((request_the_dragon.text))
        # Click on everything good
        for item_total in range(0,len(items_type)):
            if items_type[item_total] not in bad_things:
                threading.Thread(target=RequestTheDragon, args=("post", items_uuid[item_total])).start()
                test.append(items_type)
```

Nous avons les prémices permettant d'exploiter le jeu. Maintenant, il faut que nous ayons la possibilité de spammer l'API du jeu afin de cliquer automatiquement sur chaque élément dont nous avons besoin.

Tout le reste du script se trouve ici :

![exploit_the_dragon](https://github.com/PurpulHat/DG-hAck/blob/main/exploit_the_dragon.py)

Rapidement, le script aura pour but de détecter automatiquement tous les monstres qui apparaissent afin de les tuer, d'esquiver les bombes/fox et de récupérer tout le reste, c'est-à-dire les pièces.

Une fois le script lancé, nous pourrons enfin atteindre le niveau 20, gagner le succès, et obtenir le flag !
