---
layout: page
title: Merge sort
draft: true
---

## Pseudocode

```golang
func merge_sort(input []number, start uint, end uint) {
    if start < end {
        middle := floor( start + (end - start) / 2)
    }
    merge_sort(input, start, middle)
    merge_sort(input, middle, end)
    merge(input, start, middle, end)
}

func merge(input []number, start uint, middle uint, end uint) {
    lhs := make([]number, middle - start)
    lhs = append(lhs, input[start : middle])
    rhs := make([]number, end - middle)
    rhs = append(rhs, input[middle : end])
    lhs_index := 0
    rhs_index := 0
    index := 0
    for ; lhs_index < len(lhs) && rhs_index < len(rhs); {
        if lhs[lhs_index] < rhs[rhs_index] {
            input[index] = lhs[lhs_index]
            lhs_index++
        }
        else {
            input[index] = rhs[rhs_index]
            rhs_index++
        }
        index++;
    }
    rest := lhs[lhs_index :]
    if len(rest) == 0 {
        rest = rhs[rhs_index :]
    }
    for item in rhs {
        input[index] = item
        index++
    }
}
```