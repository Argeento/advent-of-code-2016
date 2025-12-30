# Advent of Code 2016

My solutions to the [AoC 2016](https://adventofcode.com/2016) challenges written in [Civet](https://civet.dev).

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

## Day 3: Squares With Three Sides ⭐⭐

```ts
triangles := getLines(input).map toNumbers
isValidTriangle := ([a, b, c]: number[]) => a + b > c
log triangles.map([...&].sort asc).filter(isValidTriangle)#
rotated := triangles |> rotateMatrixLeft |> flatten |> toChunks 3 |> .map .sort asc
log rotated.filter(isValidTriangle)#
```

## Day 4: Security Through Obscurity ⭐⭐

```ts
rooms := getLines input

moveLetter := (letter: string, number: number) =>
  charCode := (letter.charCodeAt(0) - 97 + number) %% 26
  String.fromCharCode charCode + 97

log sum for room of rooms
  sectorId := int room[-10...-7]
  name := room[...-11].replaceAll '-', ''
  checksum := countChars(name).map(.char).join ''

  if checksum.startsWith room[-6...-1]
    decrypted := [...name].map(moveLetter ., sectorId).join ''
    log sectorId if decrypted.includes 'northpole'
    sectorId
```

## Day 5: How About a Nice Game of Chess? ⭐⭐

```ts
md5 from md5

passwordA .= ''
passwordB := new Array(8).fill ''

for i of [0..]
  hash := md5 `${input.trim()}${i}`
  continue if not hash.startsWith '00000'
  passwordA += hash.5 if passwordA# < 8

  pos := +hash.5
  if pos >= 0 and pos < 8 and passwordB[pos] is ''
    passwordB[pos] = hash.6   
    break if passwordB.every & is not ''

log passwordA, passwordB.join ''
```

## Day 6: Signals and Noise ⭐⭐

```ts
cols := getLines(input).map .split '' |> rotateMatrixRight

log cols.map(countChars(.).0.char).join ''
log cols.map(countChars(.).-1.char).join ''
```

## Day 7: Internet Protocol Version 7 ⭐⭐

```ts
ips := getLines input

hasAbba := (str: string) => /(.)(?!\1)(.)\2\1/.test str
getAbas := (str: string) => [...str.matchAll /(?=((.)(?!\2)(.)\2))/g].map &.1

tls .= 0
ssl .= 0

for ip of ips
  seq := ip.split /\[.*?\]/
  hyper := [...ip.matchAll /\[(.*?)\]/g].map &.1

  tls++ if seq.some(hasAbba) and hyper.every negate hasAbba
  ssl++ if seq.flatMap(getAbas).some (aba) => hyper.some .includes aba.1 + aba.0 + aba.1

log tls, ssl
```

## Day 8: Two-Factor Authentication ⭐⭐

```ts
screen := createArray 6, 50, '.'
rotate := (line: string[], move: number) =>
  line[-move..] ++ line[...-move]

for operation of getLines input
  if operation.startsWith 'rect'
    [width, height] := toNumbers operation
    for y of [0...height]
      for x of [0...width]
        screen[y][x] = '#'
 
  if operation.startsWith 'rotate column'
    [column, move] := toNumbers operation
    newColumn := rotate getColumn(screen, column), move
    screen.forEach (row, i) => row[column] = newColumn[i]

  if operation.startsWith 'rotate row'
    [row, move] := toNumbers operation
    screen[row] = rotate screen[row], move

log screen.flat().filter(& is '#')#
printArray screen
```

## Day 9: Explosives in Cyberspace ⭐⭐

```ts
file .= input
currentIndex .= -Infinity
repeats := map input, +/[A-Z]/.test .

for marker of input.matchAll /\(\d+x\d+\)/g
  [length, times] := toNumbers marker.0
  index := marker.index + marker.0#

  if marker.index >= currentIndex
    repeated := file[index...index + length].repeat times
    file = file[...marker.index] + repeated + file[index + length...]
    currentIndex = index + length
  
  for i of [index...index + length]
    repeats[i] *= times

log file#, sum repeats
```

## Day 10: Balance Bots ⭐⭐

```ts
instructions := getLines input
inputs := instructions.filter .startsWith 'value'
rules := instructions.filter .startsWith 'bot'

bots: Record<number, number[]> := {}
outputs: Record<number, number[]> := {}

for input of inputs
  [value, bot] := toNumbers input
  (bots[bot] ?= []).push value

until values(bots).every .# is 0
  for rule of rules
    [bot, lowDest, highDest] := toNumbers rule
    continue if bots[bot]?# is not 2

    [low, high] := bots[bot].sort asc
    log bot if low is 17 and high is 61

    bots[bot] = []
    'low to bot' is in rule
      ? (bots[lowDest] ?= []).push low
      : (outputs[lowDest] ?= []).push low
    'high to bot' is in rule
      ? (bots[highDest] ?= []).push high
      : (outputs[highDest] ?= []).push high

log outputs.0.0 * outputs.1.0 * outputs.2.0
```

## Day 12: Clock Signal ⭐⭐

```ts
instructions := getLines input

function execute(cRegister: number)
  registers: Record<string, number> :=
    a: 0
    b: 0
    c: cRegister
    d: 0

  getValue := (x: string) => /\d/.test(x) ? int x : registers[x]

  for i .= 0; i < instructions#; i++
    [command, x, y] := instructions[i].split ' '
    switch command
      'cpy' registers[y] = getValue x
      'inc' registers[x] += 1
      'dec' registers[x] -= 1
      'jnz' i += -1 + int y if getValue x

  return registers

log execute(0).a, execute(1).a
```

## Day 13: A Maze of Twisty Little Cubicles ⭐⭐

```ts
UndirectedGraph from graphology
{ bidirectional } from graphology-shortest-path;

isWall := (x: number, y: number) =>
  nr := x*x + 3*x + 2*x*y + y + y*y + int input
  [...nr.toString 2].filter(+&)# % 2

graph := new UndirectedGraph

for y of [0...50]
  for x of [0...50]
    continue if isWall x, y
    graph.mergeNode `${x},${y}`
    neighbors := []
      [x+1, y]
      [x-1, y]
      [x, y+1]
      [x, y-1]

    for [nx, ny] of neighbors
      continue if isWall nx, ny
      graph.mergeNode `${nx},${ny}`
      graph.mergeEdge `${x},${y}`, `${nx},${ny}`

log bidirectional(graph, '1,1', '31,39')!length - 1

log sum flatten for y of [0...50]
  for x of [0...50]
    continue if isWall x, y
    try 
      path := bidirectional graph, '1,1', `${x},${y}`
      1 if path!length - 1 <= 50
```

## Day 14: One-Time Pad ⭐⭐

```ts
md5 from md5

getHash := memoize md5 `${input}${.}`

stretch := memoize (i: number) =>
  hash .= getHash i
  hash = md5 hash for [0...2016]
  hash

findKey := (generator: (i: number) => string) =>
  keys .= 0
  for i of [0..]
    if match := generator(i).match /(.)\1\1/
      for j of [i<..i + 1000]
        if match.1.repeat(5) is in generator j
          return i if ++keys is 64
          break

log findKey getHash
log findKey stretch
```
