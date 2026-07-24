# nol.cache

Cache mémoire **TTL / LRU** en pur [Nolc](https://github.com/Noliae-France/nolc), sans aucune dépendance. Interface calquée sur les verbes de Redis (`set`/`get`/`del`/`exists`/`setex`/`ttl`/`flushall`/`dbsize`) pour qu'un code écrit contre ce cache mémoire puisse plus tard viser un backend distant **sans changer d'API** — Redis n'est jamais obligatoire.

## Installation

Dans le `nolc.toml` de votre projet :

```toml
[dependances]
"nol.cache" = { git = "https://github.com/Noliae-France/nol-cache" }
```

## Exemple

```nol
import "nol.cache"

fn main() {
    let c = cache_neuf(1000)          // capacité max : 1000 entrées (0 = illimité)

    cache_set(c, "utilisateur:42", "Alice")   // sans expiration
    cache_setex(c, "jeton", 300, "abc...")     // expire dans 300 s

    match cache_get(c, "utilisateur:42") {
        some(v) => print("trouvé: " + v),
        none    => print("absent ou expiré"),
    }

    print(text(cache_dbsize(c)))       // nombre d'entrées vivantes
}
```

## API

| Fonction | Rôle |
|---|---|
| `cache_neuf(capacite: Int) -> Cache` | crée un cache (`capacite <= 0` = illimité) |
| `cache_poser(c, cle, valeur, ttl_secondes)` | écrit avec un TTL relatif (`0` = jamais) |
| `cache_poser_expire(c, cle, valeur, expire_epoch)` | écrit avec une expiration absolue |
| `cache_obtenir(c, cle) -> Option<Text>` | lit (applique l'expiration, rafraîchit le LRU) |
| `cache_contient(c, cle) -> Bool` | présence (hors entrées expirées) |
| `cache_ttl(c, cle) -> Int` | secondes restantes ; `-1` absente, `0` sans TTL |
| `cache_supprimer(c, cle)` | supprime une clé |
| `cache_vider(c)` | vide tout |
| `cache_taille(c) -> Int` | nombre d'entrées stockées |
| `cache_purger(c)` | retire les entrées expirées |

### Alias compatibles Redis

`cache_set`, `cache_setex`, `cache_get`, `cache_del`, `cache_exists`, `cache_flushall`, `cache_dbsize`.

## Sémantique

- **TTL** : expiration appliquée paresseusement à la lecture, plus `cache_purger` pour un balayage explicite.
- **LRU** : quand la capacité est dépassée, l'entrée la **moins récemment utilisée** est évincée. L'ordre est porté par un tick logique stocké dans chaque entrée (aucun état scalaire mutable), donc strictement monotone et correct même sous accès rapides.
- **Mémoire** : les entrées vivent dans une `Map` (sémantique de référence en Nolc) ; un `Cache` se passe librement entre fonctions.

## Licence

MIT © 2026 Bastien LANGUEDOC.
