# Cap 11 - Working with Geospatial Data

# 002 Adding GeoJSON Data
--> MongoDB permite trabalhar com dados espaciais em formato GeoJSON, um padrão que especifica a 
estrutura JSON necessária para representar objetos geográficos (pontos, linhas, polígonos, etc).
```javascript
db.places.insertOne({
    name: "right there",
    location: {
        type: "Point",
        coordinates: [ -122.47243, 37.76725 ]
    }
})
```
* Insere um documento representando um `Point` em uma determinada coordenada `[longitude, latitude]`.

# 003 Running Geo Queries
--> O operador `$near` permite confirmar a proximidade entre objetos geográficos:
```javascript
db.places.find({ location: {
    $near: {
        $geometry: { type: "Point", coordinates: [-12.4567, 53.2345] },
        $minDistance: 100,
        $maxDistance: 2000
    }
}})
```
* Consulta objetos que estão entre `100` a `2000` metros do ponto situado em `[-12.4567, 53.2345]`.

# 004 Adding a Geospatial Index to Track the Distance
--> Os operadores geográficos só funcionam em campos que possuem index geográfico, e caso o mesmo não exista, 
é retornado um erro. Indexes geográficos são criados com o tipo `2dsphere`:
```javascript
db.places.createIndex({location: "2dsphere"})
```
* Cria um index geográfico no campo `location`.

# 006 Finding Places Inside a Certain Area
--> O operador `$geoWithin` permite verificar objetos geográficos que se encontram dentro de uma área:
```javascript
db.places.find({ location: {
    $geoWithin: {
        $geometry: { type: "Polygon", coordinates: [[[1, 2], [3, 4], [5, 6], [7, 8]]] }
    }
}})
```
* Verifica os objetos do campo `location` que estão dentro de um `Polygon` informado.

# 007 Finding Out If a User Is Inside a Specific Area
--> O operador `$geoIntersects` permite verificar objetos geográficos que fazem intersecção com outro objeto 
especificado:
```javascript
db.places.find({ area: {
    $geoIntersects: {
        $geometry: { type: "Point", coordinates: [-12.4567, 53.2345] }
    }
}})
```
* Verifica os objetos do campo `area` que fazem intersecção com um `Point`.

# 008 Finding Places Within a Certain Radius
--> O operador `$centerSphere` permite representar uma esfera desenhada com um raio partindo de uma coordenada. 
E pode ser utilizada em conjunto com `$geoWithin`:
```javascript
db.places.find({ location: {
    $geoWithin: {
        $centerShere: [[-122.46203, 37.77286], 1 / 6478.1]
    }
}})
```
* Verifica os objetos do campo `location` que estão dentro de uma esfera.
* `1 / 6478.1` é o cálculo de `1` quilômetro para radianos.