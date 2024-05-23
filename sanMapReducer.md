1. Liste de tous les livres (type "Book"):
```
db.publis.find({type: "Book"})
```

2. Liste des publications depuis 2011 :
```
db.publis.find({year: {$gte: 2011}})
```

3. Liste des livres depuis 2014 :
```
db.publis.find({type: "Book", year: {$gte: 2014}})
```

4. Liste des publications de l'auteur "Toru Ishida" :
```
db.publis.find({authors: "Toru Ishida"})
```

5. Liste de tous les éditeurs distincts :
```
db.publis.distinct("publisher")
```

6. Liste de tous les auteurs distincts :
```
db.publis.distinct("authors")
```

7. Trier les publications de "Toru Ishida" par titre de livre et par page de début :
```
db.publis.find({authors: "Toru Ishida"}).sort({title: 1, pages: 1})
```

8. Projeter le résultat sur le titre de la publication et les pages :
```
db.publis.find({authors: "Toru Ishida"}, {title: 1, pages: 1, _id: 0})
```

9. Compter le nombre de ses publications :
```
db.publis.find({authors: "Toru Ishida"}).count()
```

10. Compter le nombre de publications depuis 2011 et par type :

```
db.publis.aggregate([
  {$match: {year: {$gte: 2011}}},
  {$group: {_id: "$type", count: {$sum: 1}}}
])
```

11. Compter le nombre de publications par auteur et trier le résultat par ordre croissant :
```
db.publis.aggregate([
  {$unwind: "$authors"},
  {$group: {_id: "$authors", count: {$sum: 1}}},
  {$sort: {count: 1}}
])
```