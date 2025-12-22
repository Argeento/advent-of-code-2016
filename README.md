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
