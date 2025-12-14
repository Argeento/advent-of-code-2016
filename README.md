# Advent of Code 2025

My solutions to the [AoC 2025](https://adventofcode.com/2025) challenges written in [Civet](https://civet.dev).

## Day 1: No Time for a Taxicab ⭐⭐

```ts
dirs := input.split(', ').map [&.0, +&[1..]] as const
player := x: 0, y: 0, dir: 0

visited := new Set
firstVisited: Point | null .= null

for [dir, steps] of dirs
  player.dir += dir is 'R' ? 1 : -1
  player.dir %%= 4

  for _ of [0...steps]
    player.x += 1 if player.dir is 1
    player.x -= 1 if player.dir is 3
    player.y += 1 if player.dir is 2
    player.y -= 1 if player.dir is 0

    pointKey := `${player.x},${player.y}`
    if firstVisited is null and visited.has pointKey
      firstVisited = x: player.x, y: player.y
    visited.add pointKey

log getManhattanDistance player
log getManhattanDistance firstVisited!
```

## Day 2: Bathroom Security ⭐⭐

```ts
pad1 := ['123', '456', '789']
pad2 := ['  1', ' 234', '56789', ' ABC', '  D']
current1 := x: 1, y: 1
current2 := x: 0, y: 2
code1 .= ''
code2 .= ''

for line of getLines input
  for dir of line
    if dir is 'U'
      current2.y -= 1 if pad2[current2.y - 1]?[current2.x]?trim()
      current1.y -= 1 if pad1[current1.y - 1]?[current1.x]?trim()
    if dir is 'D'
      current2.y += 1 if pad2[current2.y + 1]?[current2.x]?trim()
      current1.y += 1 if pad1[current1.y + 1]?[current1.x]?trim()
    if dir is 'L'
      current2.x -= 1 if pad2[current2.y][current2.x - 1]?trim()
      current1.x -= 1 if pad1[current1.y][current1.x - 1]?trim()
    if dir is 'R'
      current2.x += 1 if pad2[current2.y][current2.x + 1]?trim()
      current1.x += 1 if pad1[current1.y][current1.x + 1]?trim()
  code1 += pad1[current1.y][current1.x]
  code2 += pad2[current2.y][current2.x]

log code1, code2
```
