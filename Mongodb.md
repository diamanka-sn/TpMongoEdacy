### 1. Pour chaque document de type livre, émettre le document avec pour clé « title »

function map() {
    if (this.type === "Book") {
        emit(this.title, this);
    }
}

function reduce(key, values) {
    return values[0];
}


### 2. Pour chacun de ces livres, donner le nombre de ses auteurs


function map() {
    if (this.type === "Book") {
        emit(this.title, { nombres: this.authors.length });
    }
}

function reduce(key, values) {
    return values[0];
}

### 3. Pour chaque livre publié par Springer et composé de chapitres (ayant l’attribut « booktitle »), donner le nombre des chapitres

function map() {
    if (this.publisher === "Springer" && this.booktitle) {
        emit(this._id, { nombreChapitre: 1 });
    }
}

function reduce(key, values) {
    return { nombreChapitre: values.length };
}

function map() {
    if (this.publisher === "Springer") {
        emit(this.year, { count: 1 });
    }
}

function reduce(key, values) {
    return { count: values.length };
}

### 5. Pour chaque couple « publisher & année », donner le nombre de publications


function map() {
    if (this.publisher) {
        emit({ publisher: this.publisher, year: this.year }, { count: 1 });
    }
}

function reduce(key, values) {
    return { count: values.length };
}

### 6. Pour l’auteur « Toru Ishida », donner le nombre de publications par année

function map() {
    if (this.authors.includes("Toru Ishida")) {
        emit(this.year, { count: 1 });
    }
}

function reduce(key, values) {
    return { count: values.length };
}

### 7. Pour l’auteur « Toru Ishida », donner le nombre moyen de pages pour ses articles (type Article)

function map() {
    if (this.authors.includes("Toru Ishida") && this.type === "Article") {
        let pages = this.pages.end - this.pages.start + 1;
        emit(this._id, { pages: pages, count: 1 });
    }
}

function reduce(key, values) {
    let totalPages = values.reduce((sum, val) => sum + val.pages, 0);
    let totalCount = values.reduce((sum, val) => sum + val.count, 0);
    return { averagePages: totalPages / totalCount };
}

### 8. Pour chaque auteur, lister les titres de ses publications

function map() {
    for (let author of this.authors) {
        emit(author, { titles: [this.title] });
    }
}

function reduce(key, values) {
    let titles = [];
    values.forEach(value => titles = titles.concat(value.titles));
    return { titles: titles };
}

### 9. Pour chaque auteur, lister le nombre de publications associé à chaque année

function map() {
    for (let author of this.authors) {
        emit(author, { year: this.year, count: 1 });
    }
}

function reduce(key, values) {
    let result = {};
    values.forEach(value => {
        if (!result[value.year]) {
            result[value.year] = 0;
        }
        result[value.year] += value.count;
    });
    return result;
}

### 10. Pour l’éditeur « Springer », donner le nombre d’auteurs par année

function map() {
    if (this.publisher === "Springer") {
        emit(this.year, { authors: this.authors });
    }
}

function reduce(key, values) {
    let authorSet = new Set();
    values.forEach(value => value.authors.forEach(author => authorSet.add(author)));
    return { authorCount: authorSet.size };
}

### 11. Compter les publications de plus de 3 auteurs


function map() {
    if (this.authors.length > 3) {
        emit(this._id, { count: 1 });
    }
}

function reduce(key, values) {
    return { count: values.length };
}

### 12. Pour chaque éditeur, donner le nombre moyen de pages par publication

function map() {
    if (this.pages && this.pages.start !== undefined && this.pages.end !== undefined) {
        let pages = this.pages.end - this.pages.start + 1;
        emit(this.publisher, { totalPages: pages, count: 1 });
    }
}

function reduce(key, values) {
    let totalPages = values.reduce((sum, val) => sum + val.totalPages, 0);
    let totalCount = values.reduce((sum, val) => sum + val.count, 0);
    return { averagePages: totalPages / totalCount };
}


### 13. Pour chaque auteur, donner le minimum et le maximum des années avec des publications, ainsi que le nombre total de publications

function map() {
    for (let author of this.authors) {
        emit(author, { year: this.year, count: 1 });
    }
}

function reduce(key, values) {
    let minYear = Math.min(...values.map(v => v.year));
    let maxYear = Math.max(...values.map(v => v.year));
    let totalCount = values.length;
    return { minYear: minYear, maxYear: maxYear, totalCount: totalCount };
}


#### Suite du TP

### Mettre à jour tous les livres contenant "database" en ajoutant l'attribut "Genre" : "Database"
    avant 
    db.publis_copy.find({title: /database/i, type: "Book"}, {title: 1, type: 1, genre: 1})
    Mise à jour
    db.publis_copy.updateMany({title: /database/i, type: "Book"}, {$set: {genre: "Database"}})
    après
    db.publis_copy.find({title: /database/i, type: "Book"}, {title: 1, type: 1, genre})

### Supprimer le champ "number" de tous les articles
    avant et aprés
    db.publis_copy.find({type: "Article"}, {title: 1, number: 1})
    Mise à jour 
    db.publis_copy.updateMany({type: "Article"}, {$unset: {number: 1}})

### Supprimer tous les articles n'ayant aucun auteur
    avant et aprés
    db.publis_copy.find({type: "Article", $or: [{authors: []}, {authors: {$exists: false}}]})
    Mise à jour
    db.publis_copy.deleteMany({type: "Article", $or: [{authors: []}, {authors: {$exists: false}}]})

### Modifier toutes les publications ayant des pages pour ajouter le champ "pp" avec pour valeur le motif suivant: pages.start--pages.end
    avant et aprés
    db.publis_copy.find({pages: {$exists: true}}, {title: 1, pages: 1, pp: 1})
    Mise à jour
    db.publis_copy.updateMany({pages: {$exists: true}},{$set: {pp: {$concat: ["$pages.start", "--", {$toString: "$pages.end"}]}}})


db.publis_copy.updateMany(
  { pages: { $exists: true } },
  { $set: { pp: { $concat: [ { $toString: "$pages.start" }, "--", { $toString: "$pages.end" } ] } } }
)