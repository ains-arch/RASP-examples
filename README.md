# Functions in RASP

An s-op that contains the indices in reverse order.
```
>> reverse_indices = aggregate(select(indices, length-indices-1, ==), indices);
     s-op: reverse_indices
 	    Example: reverse_indices("hello") = [4, 3, 2, 1, 0] (ints)
>> reverse_indices("hello");
	 =  [4, 3, 2, 1, 0] (ints)
```

A function that "rotates" the input text by the specified number of characters.
```
>> def rotate(seq,n) {
        return aggregate(
            select(indices,indices+length-n,==) or select(indices+n,indices,==), tokens);
            }
>> rotate(tokens, 0)("hello");
	 =  [h, e, l, l, o] (strings)
>> rotate(tokens, 1)("hello");
	 =  [o, h, e, l, l] (strings)
>> rotate(tokens, 2)("hello");
	 =  [l, o, h, e, l] (strings)
```

A function that takes a sequence as input and "swaps every letter with its neighbor".
Specifically, for every even index $i$, positions $i$ and $i+1$ will be swapped;
if the length of the sequence is odd, then the last element does not move.

```
>> def miniswap(seq) {
    return aggregate(select(indices, indices + 1, ==), seq, "z")
        if indices % 2 == 0
        else aggregate(select(indices, indices-1, ==), seq, "z");
        }
>> def swap(seq) {
    return aggregate(select(indices, indices, ==), seq, "z")
    if length % 2 == 1 and indices == length - 1
    else miniswap(seq);
    }
>> swap(tokens)("hello");
         =  [e, h, l, l, o] (strings)
>> swap(tokens)("ababab");
         =  [b, a, b, a, b, a] (strings)
```

Function that returns the maximum value in the sequence repeated for every position.
```
def maxseq(seq) {
    select_earlier_in_sorted = 
        select(seq,seq,<) or (select(seq,seq,==) and select(indices,indices,<));
    target_position = 
        selector_width(select_earlier_in_sorted);
    select_new_val = 
        select(target_position,indices,==);
    return aggregate(select_new_val,seq)[-1];
}
>> maxseq(tokens)("ababcabab");
	 =  [c]*9 (strings)
```

A function that performs sequence reversal "autogeneratively".
That is, it will take a sequence as input, and the sequence will contain a special token `$` that marks the "end of the prompt".
The text before the `$` is unchanged, and the text after the `$` is the text before the `$` reversed (this text represents the model's response to the prompt).
The code is robust to the case when the length of text after `$` is not the same as the length of text before `$`.

```
def reverse_ag(seq) {
    dollar_sign_index = aggregate(select_from_first(seq, "$"), indices);
    _dollar_selector = select(indices, dollar_sign_index, <=);
    return aggregate((_dollar_selector and select(indices, indices, ==))
        or (_dollar_selector and select(indices, dollar_sign_index*2 - indices, ==)), seq, "");
}
>> reverse_ag(tokens)("hello$     ");
	 =  [h, e, l, l, o, $, o, l, l, e, h] (strings)
>> reverse_ag(tokens)("hello$ ");
	 =  [h, e, l, l, o, $, o] (strings)
>> reverse_ag(tokens)("hello$X");
	 =  [h, e, l, l, o, $, o] (strings)
>> reverse_ag(tokens)("hello$XXXXXXXXXX");
	 =  [h, e, l, l, o, $, o, l, l, e, h, , , , , ] (strings)
```

A function that counts the number of times a certain token appears in the input sequence.
```
>> def howmany(seq, t) {
        return count(seq, t);
        }
>> howmany(tokens, "a")("hello");
	 =  [0]*5 (ints)
>> howmany(tokens, "h")("hello");
	 =  [1]*5 (ints)
>> howmany(tokens, "l")("hello");
	 =  [2]*5 (ints)
```

A function that counts the number of times a certain token has appeared in the input sequence so far.
```
def howmanysofar(seq, n) {
    return round((indices+1) * aggregate(select(indices, indices, <=), indicator(seq==n)));
    }
>> howmanysofar(tokens, "a")("hello");
	 =  [0]*5 (ints)
>> howmanysofar(tokens, "h")("hello");
	 =  [1]*5 (ints)
>> howmanysofar(tokens, "e")("hello");
	 =  [0, 1, 1, 1, 1] (ints)
>> howmanysofar(tokens, "l")("hello");
	 =  [0, 0, 1, 2, 2] (ints)
```


A function that performs sequence sorting "autogeneratively".
The text before the `$` is unchanged, and the text after the `$` is the text before the `$` sorted with an insertion-sort-like $O(n^2)$ sorting algorithm.
The code is robust to the case when the length of text after `$` is not the same as the length of text before `$`.

```
def sort_ag(seq) {
    reverse = aggregate(select(indices, length-indices-1, ==), tokens);
    dollar_sign_index = aggregate(select_from_first(seq, "$"), indices);
    _dollar_selector = select(indices, dollar_sign_index, <);
    select_earlier_in_sorted =
        (select(seq,seq,<) or (select(seq,seq,==) and select(indices,indices,<)) and _dollar_selector);
    target_position =                                                     
        selector_width(select_earlier_in_sorted);
    select_new_val = 
        select(target_position+dollar_sign_index+1 if indices < dollar_sign_index else target_position-1,indices,==);
    return aggregate((_dollar_selector and select(indices, indices, ==))
        or (_dollar_selector and select_new_val) or (select(tokens, "$", ==) and select("$", tokens,
        ==)), seq, "");
}
    console function: sort_ag(seq)
>> sort_ag(tokens)("hello$     ");
	 =  [h, e, l, l, o, $, e, h, l, l, o] (strings)
>> sort_ag(tokens)("hello$ ");
	 =  [h, e, l, l, o, $, e] (strings)
>> sort_ag(tokens)("hello$X");
	 =  [h, e, l, l, o, $, e] (strings)
>> sort_ag(tokens)("hello$XXXXXXXXXX");
	 =  [h, e, l, l, o, $, e, h, l, l, o, , , , , ] (strings)
```
