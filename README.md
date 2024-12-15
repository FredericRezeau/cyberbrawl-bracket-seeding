# cyberbrawl-bracket-seeding

This repository contains the official bracket seeding algorithm for [CyberBrawl.io](https://cyberbrawl.io) tournaments. The goal is to provide a deterministic and transparent way to shuffle and seed players in CyberBrawl's competitive events.

## How It Works

When creating tournament brackets for CyberBrawl.io, we follow this transparent process:

- A unique `salt` is generated, and its `SHA-256` hash is published at the time of the tournament announcement.
- Once player registrations are finalized, a `seed` is created by combining the player IDs, respecting enlistment order, with the `salt`.
- Players are shuffled using the deterministic **Fisher-Yates shuffle** algorithm.
- If the total number of players is not a power of 2, "BYE" slots are added as placeholders to complete the bracket.
- Initial matches are then determined by systematically pairing players from the two halves of the shuffled list.

This simple process ensures that the brackets are randomized, fair, and resistant to manipulation from both players and organizers.

## Process Breakdown

1. **Input:**
   - `players`: list of player IDs.
   - `salt`: predefined unique string.

2. **Steps:**
   1. Computing the `seed`:
      - Concatenate all `players` IDs into a string.
      - Append `salt` to the concatenated string.
      - Generate a SHA-256 hash of the combined string.
      - Extract the first 4 bytes of the hash and convert them to integer (big-endian).
   2. Shuffling the `players`:
      - Use the Fisher-Yates shuffle algorithm.
      - Seed the RNG with the computed `seed`.
   4. Padding the shuffled `players`:
      - If the number of players is not a power of 2, pad the shuffled `players` list with `bye` until the total length matches the nearest power of 2.
   5. Pairing the shuffled `players`:
      - Pair players from the two halves of the shuffled list.

## Example Code
```javascript
  const seedBracket = (players, salt) => {
      const padded = [...players];
      const seed = parseInt(crypto.createHash('sha256').update(padded.join('') + salt).digest('hex').slice(0, 8), 16);
      const shuffle = (array, seed) => {
          const rng = ((s) => () => (s = (s * 9301 + 49297) % 233280) / 233280)(seed);
          for (let i = array.length - 1; i > 0; i--) {
              const j = Math.floor(rng() * (i + 1));
              [array[i], array[j]] = [array[j], array[i]];
          }
          return [
              ...array,
              ...Array((2 ** Math.ceil(Math.log2(players.length))) - players.length).fill('bye')
          ];
      };
      const shuffled = shuffle(padded, seed);
      return { seed, matches: Array.from({ length: shuffled.length / 2 }, (_, i) => [shuffled[i], shuffled[shuffled.length / 2 + i]]) };
  };
```

Run it in CodeSandbox:

[![Edit on CodeSandbox](https://codesandbox.io/static/img/play-codesandbox.svg)](https://codesandbox.io/s/6p3szn)

## Official Cyberbrawl Tournaments
Here is a record of salts and players data used for seeding official tournaments. You can verify the seeding process against these values:

### Beta Launch Tournament
```json
{
    "players": [ "gonz0", "9x1r1s", "ihgux3", "2jw8kp", "vqnl28", "j52nl0", "kside", "q0vbp1", "m268p0", "master", "fo2xo6", "energy", "xqcaax", "garden", "zero", "joegkl",     
"jdtgw9", "sqcleg", "y2g1mv", "3oblfz", "eesphk", "zkneug", "641m9e", "os1ugx", "dragon", "i7ge2a", "vuhyms", "q8yoqn", "6pqirj","kunkun","hwamni"],
    "salt": "b3d10071-a0f1-e368-abc0-74db4a61b028",
    "hash": "6d1451adda4f44d3a1a27118388ea34db5c6c8316141378bd9467d4d64644c3e",
}
```

## License
MIT License
