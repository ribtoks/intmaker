---
title: Generation of spiral matrices
date: 2014-01-30T20:55:20+00:00
author: "Taras Kushnir"
permalink: /generating-spiral-matrices/
categories:
  - Programming
  - Ruby
keywords:
  - array
  - matrix
  - numbers
  - ruby
  - spiral
---
Have you ever heard about this simple problem? Generating spiral matrices. The one like this:

```
1  2  3  4  5
16 17 18 19 6
15 24 25 20 7
14 23 22 21 8
13 12 11 10 9
```

After a hour of trying different approaches (except one which consists of 4 loops for filling up & down row and left & right) I've written a working ruby snippet (just for fun):

```ruby
def spiral(n)
    arr = Array.new(n){Array.new(n){0}}
    diff = [[0, 1], [1, 0], [0, -1], [-1, 0]]
    i, j = 0, 0
    dindex = 0
    turns = 0
    curr_square = n
    shift = 0

    (n*n).times do |index|
        arr[i][j] = index + 1

        if index == (shift + 4*curr_square - 5)
            curr_square -= 2
            turns, dindex = 0, 0
            shift = index + 1
            j += 1
            next
        end

        i += diff[dindex][0]
        j += diff[dindex][1]

        if index == (shift + (turns + 1)*(curr_square - 1) - 1)
            turns += 1
            dindex += 1
        end
    end

    arr
end

arr = spiral(10)
puts arr.map{|ia| ia.join(' ')}.join("\n")
```

You can find code about on <a href="https://gist.github.com/Ribtoks/8712591" target="_blank">Gist</a>.
