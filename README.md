problem 1
```
>> reverse_indices = aggregate(select(indices, length-indices-1, ==), indices);
     s-op: reverse_indices
 	 Example: reverse_indices("hello") = [4, 3, 2, 1, 0] (ints)
>> reverse_indices("hello")
.. ;
	 =  [4, 3, 2, 1, 0] (ints)
```

problem 2
```
>> def rotate(seq,n) {
..   return aggregate(
..   select(indices,indices+length-n,==) or select(indices+n,indices,==), tokens);
..   }
     console function: rotate(seq, n)
>> rotate(tokens, 0)("hello");
	 =  [h, e, l, l, o] (strings)
>> rotate(tokens, 1)("hello");
	 =  [o, h, e, l, l] (strings)
>> rotate(tokens, 2)("hello");
	 =  [l, o, h, e, l] (strings)
```

```
def swap(seq) {
    _selector = select(indices-1 if indices%2 else indices+1, indices, ==) or (select(indices, length-1, ==) and select(length-1, indices, ==))
        if length%2
        else _selector = select(indices-1 if indices%2 else indices+1, indices, ==), seq);
    return aggregate(_selector, tokens);
    }
```

```
def swap(seq) {
    return aggregate(select(indices-1 if indices%2 else indices+1, indices, ==), tokens)
        if length % 2 == 1
        else aggregate(select(indices-1 if indices%2 else indices+1, indices, ==)
            + (select(indices, length-1, ==) and select(length-1, indices, ==)),
            tokens);
            }
```

select(indices-1, ==) if indices%2 else select(indices+1, ==)

right shift selector and left shift selector



def swap(seq) {
    _selector = select(indices-1 if indices%2 else indices+1, indices, ==)
        if length%2 == 0
        else select(indices-1 if indices%2 else indices+1, indices, ==)
            or (select(indices, length-1, ==) and select(length-1, indices, ==))
    return aggregate(
    or
        (select(indices, length-1, ==) and select(length-1, indices, ==))
            if length%2 == 1
            else
            select(indices, tokens, !=),
        tokens);
    }
        

    if length%2 == 0
    else aggregate(select(indices, indices, ==), tokens);
    }

```
>> def miniswap(seq) {
..       return aggregate(select(indices, indices + 1, ==), seq, "z") 
..       if indices % 2 == 0
..       else aggregate(select(indices, indices-1, ==), seq, "z");
..   }
     console function: miniswap(seq)
>> def swap(seq) {
..      return aggregate(select(indices, indices, ==), seq, "z")
..      if length % 2 == 1 and indices == length - 1
..      else miniswap(seq);
..   }
     console function: swap(seq)
>> swap(tokens)("hello");
         =  [e, h, l, l, o] (strings)
>> swap(tokens)("ababab");
         =  [b, a, b, a, b, a] (strings)
```


def select_next_larger(seq) {
	select_prev_larger = 
		select(seq,seq,>) and select(indices,indices,<);
	num_prev_larger = 
		selector_width(select_prev_larger);
	return select(seq,seq,>) and select(num_prev_larger,num_prev_larger+1,==);
}

def sorter(seq) {
    select_earlier_in_sorted = 
        select(seq,seq,<) or (select(seq,seq,==) and select(indices,indices,<));
    target_position = 
        selector_width(select_earlier_in_sorted);
    select_new_val = 
        select(target_position,indices,==);
    return aggregate(select_new_val,seq)[-1][0];
}

select_earlier_in_sorted = 
        select(tokens,tokens,<) or (select(tokens,tokens,==) and select(indices,indices,<));
    target_position = 
        selector_width(select_earlier_in_sorted);
    select_new_val = 
        select(target_position,indices,==);

```
>> def sorter(seq) {
      select_earlier_in_sorted =
          select(seq,seq,<) or (select(seq,seq,==) and select(indices,indices,<));
      target_position =                                                     
          selector_width(select_earlier_in_sorted);
      select_new_val = 
          select(target_position,indices,==);                                                     
      return aggregate(select_new_val,seq)[-1][0]; 
  }
     console function: sorter(seq)
>> sorter(tokens)("ababcabab");
	 =  [c]*9 (strings)
```

```
# find the $
dollar_matrix = select_from_first(tokens, "$")
dollar_list = aggregate(dollar, indices);
select(dollar, dollar, <=)
# selector up to the $ / that gets rid of the stuff after it
# selector that's empty up to $ and reverse afterwards
# add em together
# aggregate

reverse = aggregate(select(indices, length-indices-1, ==), tokens);
def reverse_ag(seq) {
    reverse = aggregate(select(indices, length-indices-1, ==), tokens);
    dollar_sign_index = aggregate(select_from_first(seq, "$"), indices);
    _dollar_selector = select(indices, dollar_sign_index, <=);
    return aggregate((_dollar_selector and select(indices, indices, ==))
        or (_dollar_selector and select(indices, dollar_sign_index*2 - indices, ==)), seq, "");
}

>> def howmany(seq, t) {
..   return count(seq, t);
..   }
     console function: howmany(seq, t)
>> howmany(tokens, "a")("hello");
	 =  [0]*5 (ints)
>> howmany(tokens, "h")("hello");
	 =  [1]*5 (ints)
>> howmany(tokens, "l")("hello");
	 =  [2]*5 (ints)

def howmanysofar(seq, n) {
    return round((indices+1) * aggregate(select(indices, indices, <=), indicator(seq==n)));
    }

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
