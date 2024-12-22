+++
date = '2023-11-17T16:00:00-06:00'
draft = false
title = 'sed (part 2)'
tags = ['posix', 'how-to']
+++

## overview

[last](/posts/sed) time we discussed the command line utility, `sed`. as a
stream editor, `sed` provides a programmatic way to edit files or streams of
data. rather than opening up a full-fledged, interactive text editor or ide you
can reach for `sed`.

this is particularily useful when you need to make repetative edits over many
lines, or only on particular lines. `sed` is also useful for general purpose use
cases when viewing parts of files or trimming and cleaning up data.

as promised, this article will be primarily examples showing off `sed` in
different scenarios with light instructional details where necessary.

## examples

### replicating head/tail

you may have heard of the complimentary tools,
[head](https://man7.org/linux/man-pages/man1/head.1.html) and
[tail](https://man7.org/linux/man-pages/man1/tail.1.html) in order to view
select number of lines in a file.

the following prints the first 5 lines of a file[^1]

```bash
head -n5 moby_dick.txt
```

```
call me ishmael. some years ago—never mind how long precisely—having little or no money in my purse, and nothing particular to interest me on shore, i thought i would sail about a little and see the watery part of the world. it is a way i have of driving off the spleen and regulating the circulation. whenever i find myself growing grim about the mouth; whenever it is a damp, drizzly november in my soul; whenever i find myself involuntarily pausing before coffin warehouses, and bringing up the rear of every funeral i meet; and especially whenever my hypos get such an upper hand of me, that it requires a strong moral principle to prevent me from deliberately stepping into the street, and methodically knocking people’s hats off—then, i account it high time to get to sea as soon as i can. this is my substitute for pistol and ball. with a philosophical flourish cato throws himself upon his sword; i quietly take to the ship. there is nothing surprising in this. if they but knew it, almost all men in their degree, some time or other, cherish very nearly the same feelings towards the ocean with me.

there now is your insular city of the manhattoes, belted round by wharves as indian isles by coral reefs—commerce surrounds it with her surf. right and left, the streets take you waterward. its extreme downtown is the battery, where that noble mole is washed by waves, and cooled by breezes, which a few hours previous were out of sight of land. look at the crowds of water-gazers there.

circumambulate the city of a dreamy sabbath afternoon. go from corlears hook to coenties slip, and from thence, by whitehall, northward. what do you see?—posted like silent sentinels all around the town, stand thousands upon thousands of mortal men fixed in ocean reveries. some leaning against the spiles; some seated upon the pier-heads; some looking over the bulwarks of ships from china; some high aloft in the rigging, as if striving to get a still better seaward peep. but these are all landsmen; of week days pent up in lath and plaster—tied to counters, nailed to benches, clinched to desks. how then is this? are the green fields gone? what do they here?
```

the same can be done with `sed` using a numeric address range:

```bash
sed '5q' moby_dick.txt
```

```
call me ishmael. some years ago—never mind how long precisely—having little or no money in my purse, and nothing particular to interest me on shore, i thought i would sail about a little and see the watery part of the world. it is a way i have of driving off the spleen and regulating the circulation. whenever i find myself growing grim about the mouth; whenever it is a damp, drizzly november in my soul; whenever i find myself involuntarily pausing before coffin warehouses, and bringing up the rear of every funeral i meet; and especially whenever my hypos get such an upper hand of me, that it requires a strong moral principle to prevent me from deliberately stepping into the street, and methodically knocking people’s hats off—then, i account it high time to get to sea as soon as i can. this is my substitute for pistol and ball. with a philosophical flourish cato throws himself upon his sword; i quietly take to the ship. there is nothing surprising in this. if they but knew it, almost all men in their degree, some time or other, cherish very nearly the same feelings towards the ocean with me.

there now is your insular city of the manhattoes, belted round by wharves as indian isles by coral reefs—commerce surrounds it with her surf. right and left, the streets take you waterward. its extreme downtown is the battery, where that noble mole is washed by waves, and cooled by breezes, which a few hours previous were out of sight of land. look at the crowds of water-gazers there.

circumambulate the city of a dreamy sabbath afternoon. go from corlears hook to coenties slip, and from thence, by whitehall, northward. what do you see?—posted like silent sentinels all around the town, stand thousands upon thousands of mortal men fixed in ocean reveries. some leaning against the spiles; some seated upon the pier-heads; some looking over the bulwarks of ships from china; some high aloft in the rigging, as if striving to get a still better seaward peep. but these are all landsmen; of week days pent up in lath and plaster—tied to counters, nailed to benches, clinched to desks. how then is this? are the green fields gone? what do they here?
```

the `q` function immediately quits the execution of `sed`. so this in effect
says, "do the default 'print' function for all lines, and once line 5 is
processed, quit."

we could achieve the same result inversely by using `sed`'s `p` function and the
`-n` flag to disable the default print:

```bash
sed -n '1,5 p' moby_dick.txt
```

```
call me ishmael. some years ago—never mind how long precisely—having little or no money in my purse, and nothing particular to interest me on shore, i thought i would sail about a little and see the watery part of the world. it is a way i have of driving off the spleen and regulating the circulation. whenever i find myself growing grim about the mouth; whenever it is a damp, drizzly november in my soul; whenever i find myself involuntarily pausing before coffin warehouses, and bringing up the rear of every funeral i meet; and especially whenever my hypos get such an upper hand of me, that it requires a strong moral principle to prevent me from deliberately stepping into the street, and methodically knocking people’s hats off—then, i account it high time to get to sea as soon as i can. this is my substitute for pistol and ball. with a philosophical flourish cato throws himself upon his sword; i quietly take to the ship. there is nothing surprising in this. if they but knew it, almost all men in their degree, some time or other, cherish very nearly the same feelings towards the ocean with me.

there now is your insular city of the manhattoes, belted round by wharves as indian isles by coral reefs—commerce surrounds it with her surf. right and left, the streets take you waterward. its extreme downtown is the battery, where that noble mole is washed by waves, and cooled by breezes, which a few hours previous were out of sight of land. look at the crowds of water-gazers there.

circumambulate the city of a dreamy sabbath afternoon. go from corlears hook to coenties slip, and from thence, by whitehall, northward. what do you see?—posted like silent sentinels all around the town, stand thousands upon thousands of mortal men fixed in ocean reveries. some leaning against the spiles; some seated upon the pier-heads; some looking over the bulwarks of ships from china; some high aloft in the rigging, as if striving to get a still better seaward peep. but these are all landsmen; of week days pent up in lath and plaster—tied to counters, nailed to benches, clinched to desks. how then is this? are the green fields gone? what do they here?
```

the added benefit of using `sed` here is you can provide a range that doesn't
start at the beginning of the file:

```bash
sed -n '5,9 p' moby_dick.txt
```

```
circumambulate the city of a dreamy sabbath afternoon. go from corlears hook to coenties slip, and from thence, by whitehall, northward. what do you see?—posted like silent sentinels all around the town, stand thousands upon thousands of mortal men fixed in ocean reveries. some leaning against the spiles; some seated upon the pier-heads; some looking over the bulwarks of ships from china; some high aloft in the rigging, as if striving to get a still better seaward peep. but these are all landsmen; of week days pent up in lath and plaster—tied to counters, nailed to benches, clinched to desks. how then is this? are the green fields gone? what do they here?

but look! here come more crowds, pacing straight for the water, and seemingly bound for a dive. strange! nothing will content them but the extremest limit of the land; loitering under the shady lee of yonder warehouses will not suffice. no. they must get just as nigh the water as they possibly can without falling in. and there they stand—miles of them—leagues. inlanders all, they come from lanes and alleys, streets and avenues—north, east, south, and west. yet here they all unite. tell me, does the magnetic virtue of the needles of the compasses of all those ships attract them thither?

once more. say you are in the country; in some high land of lakes. take almost any path you please, and ten to one it carries you down in a dale, and leaves you there by a pool in the stream. there is magic in it. let the most absent-minded of men be plunged in his deepest reveries—stand that man on his legs, set his feet a-going, and he will infallibly lead you to water, if water there be in all that region. should you ever be athirst in the great american desert, try this experiment, if your caravan happen to be supplied with a metaphysical professor. yes, as every one knows, meditation and water are wedded for ever.
```

### replicating grep

[grep](/posts/grep) can also easily be replaced with a similar construction by
using `-n`, `p` and a regex address:

```bash
sed -n '/whale/ p' moby_dick.txt
```

```
chief among these motives was the overwhelming idea of the great whale himself. such a portentous and mysterious monster roused all my curiosity. then the wild and distant seas where he rolled his island bulk; the undeliverable, nameless perils of the whale; these, with all the attending marvels of a thousand patagonian sights and sounds, helped to sway me to my wish. with other men, perhaps, such things would not have been inducements; but as for me, i am tormented with an everlasting itch for things remote. i love to sail forbidden seas, and land on barbarous coasts. not ignoring what is good, i am quick to perceive a horror, and could still be social with it—would they let me—since it is but well to be on friendly terms with all the inmates of the place one lodges in.
by reason of these things, then, the whaling voyage was welcome; the great flood-gates of the wonder-world swung open, and in the wild conceits that swayed me to my purpose, two and two there floated into my inmost soul, endless processions of the whale, and, mid most of them all, one grand hooded phantom, like a snow hill in the air.
```

you can replicate grep's `-o` flag to print only the matching part of a regex as
well. it is a bit more complicated but it will help to show a broader range of
`sed`'s features:

```bash
sed -e -n 's/^.*(whal(e|ing)).*$/\1/g p' moby_dick.txt
```

```
whaling
whaling
whale
whale
```

ok, that was a mouthful. the gist is that we match the whole line and substitute
that with just our search term using a capture group. let's break the command
down into its constituent parts.

the `-e` flag to `sed` enables extended (modern) regular expressions[^2]
allowing for the alternative `|` regex character in the substitution:
`whal(e|ing)`. the `^` matches the beginning of the line, and the `$` matches
the end. putting it all together, the regex matches the whole line, but creates
a capture group around _only_ the word with alternatives for whale and whaling.

the second argument to the substitution is the replacement text, `\1`. this is
standard regex syntax and means, the contents of the first capture group. the
effect of this substitution is to replace each entire line that contains our
regex with just what was captured. because we ran `sed` with the `-n` flag,
nothing gets printed by default. we specify `p` as the second function to run
after the substitute in order to print lines that the first substitute matched.

you could turn this into a reusable script if you wanted by replacing the
contents of the capture group with the argument fed into your script:

```bash
function mygrep_o() {
  sed -e -n 's/^.*('$1').*$/\1/g p' "$2"
}

mygrep_o 'whal(e|ing)' moby_dick.txt
```

```
whaling
whaling
whale
whale
```

### cleaning up files

rather than trying to replicate other commands, let's do something a little
different. in our examples, the text has a blank line between each line of text.
we can delete these pretty easily by using the `d` command which stands for
`delete`.

```bash
sed '/^$/ d' moby_dick.txt
```

```
call me ishmael. some years ago—never mind how long precisely—having little or no money in my purse, and nothing particular to interest me on shore, i thought i would sail about a little and see the watery part of the world. it is a way i have of driving off the spleen and regulating the circulation. whenever i find myself growing grim about the mouth; whenever it is a damp, drizzly november in my soul; whenever i find myself involuntarily pausing before coffin warehouses, and bringing up the rear of every funeral i meet; and especially whenever my hypos get such an upper hand of me, that it requires a strong moral principle to prevent me from deliberately stepping into the street, and methodically knocking people’s hats off—then, i account it high time to get to sea as soon as i can. this is my substitute for pistol and ball. with a philosophical flourish cato throws himself upon his sword; i quietly take to the ship. there is nothing surprising in this. if they but knew it, almost all men in their degree, some time or other, cherish very nearly the same feelings towards the ocean with me.
there now is your insular city of the manhattoes, belted round by wharves as indian isles by coral reefs—commerce surrounds it with her surf. right and left, the streets take you waterward. its extreme downtown is the battery, where that noble mole is washed by waves, and cooled by breezes, which a few hours previous were out of sight of land. look at the crowds of water-gazers there.
circumambulate the city of a dreamy sabbath afternoon. go from corlears hook to coenties slip, and from thence, by whitehall, northward. what do you see?—posted like silent sentinels all around the town, stand thousands upon thousands of mortal men fixed in ocean reveries. some leaning against the spiles; some seated upon the pier-heads; some looking over the bulwarks of ships from china; some high aloft in the rigging, as if striving to get a still better seaward peep. but these are all landsmen; of week days pent up in lath and plaster—tied to counters, nailed to benches, clinched to desks. how then is this? are the green fields gone? what do they here?
but look! here come more crowds, pacing straight for the water, and seemingly bound for a dive. strange! nothing will content them but the extremest limit of the land; loitering under the shady lee of yonder warehouses will not suffice. no. they must get just as nigh the water as they possibly can without falling in. and there they stand—miles of them—leagues. inlanders all, they come from lanes and alleys, streets and avenues—north, east, south, and west. yet here they all unite. tell me, does the magnetic virtue of the needles of the compasses of all those ships attract them thither?
once more. say you are in the country; in some high land of lakes. take almost any path you please, and ten to one it carries you down in a dale, and leaves you there by a pool in the stream. there is magic in it. let the most absent-minded of men be plunged in his deepest reveries—stand that man on his legs, set his feet a-going, and he will infallibly lead you to water, if water there be in all that region. should you ever be athirst in the great american desert, try this experiment, if your caravan happen to be supplied with a metaphysical professor. yes, as every one knows, meditation and water are wedded for ever.
but here is an artist. he desires to paint you the dreamiest, shadiest, quietest, most enchanting bit of romantic landscape in all the valley of the saco. what is the chief element he employs? there stand his trees, each with a hollow trunk, as if a hermit and a crucifix were within; and here sleeps his meadow, and there sleep his cattle; and up from yonder cottage goes a sleepy smoke. deep into distant woodlands winds a mazy way, reaching to overlapping spurs of mountains bathed in their hill-side blue. but though the picture lies thus tranced, and though this pine-tree shakes down its sighs like leaves upon this shepherd’s head, yet all were vain, unless the shepherd’s eye were fixed upon the magic stream before him. go visit the prairies in june, when for scores on scores of miles you wade knee-deep among tiger-lilies—what is the one charm wanting?—water—there is not a drop of water there! were niagara but a cataract of sand, would you travel your thousand miles to see it? why did the poor poet of tennessee, upon suddenly receiving two handfuls of silver, deliberate whether to buy him a coat, which he sadly needed, or invest his money in a pedestrian trip to rockaway beach? why is almost every robust healthy boy with a robust healthy soul in him, at some time or other crazy to go to sea? why upon your first voyage as a passenger, did you yourself feel such a mystical vibration, when first told that you and your ship were now out of sight of land? why did the old persians hold the sea holy? why did the greeks give it a separate deity, and own brother of jove? surely all this is not without meaning. and still deeper the meaning of that story of narcissus, who because he could not grasp the tormenting, mild image he saw in the fountain, plunged into it and was drowned. but that same image, we ourselves see in all rivers and oceans. it is the image of the ungraspable phantom of life; and this is the key to it all.
now, when i say that i am in the habit of going to sea whenever i begin to grow hazy about the eyes, and begin to be over conscious of my lungs, i do not mean to have it inferred that i ever go to sea as a passenger. for to go as a passenger you must needs have a purse, and a purse is but a rag unless you have something in it. besides, passengers get sea-sick—grow quarrelsome—don’t sleep of nights—do not enjoy themselves much, as a general thing;—no, i never go as a passenger; nor, though i am something of a salt, do i ever go to sea as a commodore, or a captain, or a cook. i abandon the glory and distinction of such offices to those who like them. for my part, i abominate all honorable respectable toils, trials, and tribulations of every kind whatsoever. it is quite as much as i can do to take care of myself, without taking care of ships, barques, brigs, schooners, and what not. and as for going as cook,—though i confess there is considerable glory in that, a cook being a sort of officer on ship-board—yet, somehow, i never fancied broiling fowls;—though once broiled, judiciously buttered, and judgmatically salted and peppered, there is no one who will speak more respectfully, not to say reverentially, of a broiled fowl than i will. it is out of the idolatrous dotings of the old egyptians upon broiled ibis and roasted river horse, that you see the mummies of those creatures in their huge bake-houses the pyramids.
no, when i go to sea, i go as a simple sailor, right before the mast, plumb down into the forecastle, aloft there to the royal mast-head. true, they rather order me about some, and make me jump from spar to spar, like a grasshopper in a may meadow. and at first, this sort of thing is unpleasant enough. it touches one’s sense of honor, particularly if you come of an old established family in the land, the van rensselaers, or randolphs, or hardicanutes. and more than all, if just previous to putting your hand into the tar-pot, you have been lording it as a country schoolmaster, making the tallest boys stand in awe of you. the transition is a keen one, i assure you, from a schoolmaster to a sailor, and requires a strong decoction of seneca and the stoics to enable you to grin and bear it. but even this wears off in time.
what of it, if some old hunks of a sea-captain orders me to get a broom and sweep down the decks? what does that indignity amount to, weighed, i mean, in the scales of the new testament? do you think the archangel gabriel thinks anything the less of me, because i promptly and respectfully obey that old hunks in that particular instance? who ain’t a slave? tell me that. well, then, however the old sea-captains may order me about—however they may thump and punch me about, i have the satisfaction of knowing that it is all right; that everybody else is one way or other served in much the same way—either in a physical or metaphysical point of view, that is; and so the universal thump is passed round, and all hands should rub each other’s shoulder-blades, and be content.
again, i always go to sea as a sailor, because they make a point of paying me for my trouble, whereas they never pay passengers a single penny that i ever heard of. on the contrary, passengers themselves must pay. and there is all the difference in the world between paying and being paid. the act of paying is perhaps the most uncomfortable infliction that the two orchard thieves entailed upon us. but being paid,—what will compare with it? the urbane activity with which a man receives money is really marvellous, considering that we so earnestly believe money to be the root of all earthly ills, and that on no account can a monied man enter heaven. ah! how cheerfully we consign ourselves to perdition!
finally, i always go to sea as a sailor, because of the wholesome exercise and pure air of the fore-castle deck. for as in this world, head winds are far more prevalent than winds from astern (that is, if you never violate the pythagorean maxim), so for the most part the commodore on the quarter-deck gets his atmosphere at second hand from the sailors on the forecastle. he thinks he breathes it first; but not so. in much the same way do the commonalty lead their leaders in many other things, at the same time that the leaders little suspect it. but wherefore it was that after having repeatedly smelt the sea as a merchant sailor, i should now take it into my head to go on a whaling voyage; this the invisible police officer of the fates, who has the constant surveillance of me, and secretly dogs me, and influences me in some unaccountable way—he can better answer than any one else. and, doubtless, my going on this whaling voyage, formed part of the grand programme of providence that was drawn up a long time ago. it came in as a sort of brief interlude and solo between more extensive performances. i take it that this part of the bill must have run something like this:
“grand contested election for the presidency of the united states. “whaling voyage by one ishmael. “bloody battle in affghanistan.”
though i cannot tell why it was exactly that those stage managers, the fates, put me down for this shabby part of a whaling voyage, when others were set down for magnificent parts in high tragedies, and short and easy parts in genteel comedies, and jolly parts in farces—though i cannot tell why this was exactly; yet, now that i recall all the circumstances, i think i can see a little into the springs and motives which being cunningly presented to me under various disguises, induced me to set about performing the part i did, besides cajoling me into the delusion that it was a choice resulting from my own unbiased freewill and discriminating judgment.
chief among these motives was the overwhelming idea of the great whale himself. such a portentous and mysterious monster roused all my curiosity. then the wild and distant seas where he rolled his island bulk; the undeliverable, nameless perils of the whale; these, with all the attending marvels of a thousand patagonian sights and sounds, helped to sway me to my wish. with other men, perhaps, such things would not have been inducements; but as for me, i am tormented with an everlasting itch for things remote. i love to sail forbidden seas, and land on barbarous coasts. not ignoring what is good, i am quick to perceive a horror, and could still be social with it—would they let me—since it is but well to be on friendly terms with all the inmates of the place one lodges in.
by reason of these things, then, the whaling voyage was welcome; the great flood-gates of the wonder-world swung open, and in the wild conceits that swayed me to my purpose, two and two there floated into my inmost soul, endless processions of the whale, and, mid most of them all, one grand hooded phantom, like a snow hill in the air.
```

it worked! but what if you only want to view the first few lines? we _coooould_
pipe the output of the previous script through head, but where\'s the fun in
that? this is an article on `sed` and by golly we\'re going to use `sed`!

```bash
sed -n '1,6 {
  /^$/ d
  p
}' moby_dick.txt
```

```
call me ishmael. some years ago—never mind how long precisely—having little or no money in my purse, and nothing particular to interest me on shore, i thought i would sail about a little and see the watery part of the world. it is a way i have of driving off the spleen and regulating the circulation. whenever i find myself growing grim about the mouth; whenever it is a damp, drizzly november in my soul; whenever i find myself involuntarily pausing before coffin warehouses, and bringing up the rear of every funeral i meet; and especially whenever my hypos get such an upper hand of me, that it requires a strong moral principle to prevent me from deliberately stepping into the street, and methodically knocking people’s hats off—then, i account it high time to get to sea as soon as i can. this is my substitute for pistol and ball. with a philosophical flourish cato throws himself upon his sword; i quietly take to the ship. there is nothing surprising in this. if they but knew it, almost all men in their degree, some time or other, cherish very nearly the same feelings towards the ocean with me.
there now is your insular city of the manhattoes, belted round by wharves as indian isles by coral reefs—commerce surrounds it with her surf. right and left, the streets take you waterward. its extreme downtown is the battery, where that noble mole is washed by waves, and cooled by breezes, which a few hours previous were out of sight of land. look at the crowds of water-gazers there.
circumambulate the city of a dreamy sabbath afternoon. go from corlears hook to coenties slip, and from thence, by whitehall, northward. what do you see?—posted like silent sentinels all around the town, stand thousands upon thousands of mortal men fixed in ocean reveries. some leaning against the spiles; some seated upon the pier-heads; some looking over the bulwarks of ships from china; some high aloft in the rigging, as if striving to get a still better seaward peep. but these are all landsmen; of week days pent up in lath and plaster—tied to counters, nailed to benches, clinched to desks. how then is this? are the green fields gone? what do they here?
```

well that _technically_ worked. but that leaves two questions. 1) if the address
provided was lines 1 through 6, why were only 3 lines printed? and 2) what is
going on with the curly braces?

the first question is easily enough answered; of the first 6 lines we specified
in the address range, 3 of them were blank, so though they don't show up in the
output, there were considered as we requested from the input stream.

question 2 will take a bit more explanation. in the man page, the first function
listed is the following:

```
[2addr] function-list
        execute function-list only when the pattern space is selected.
```

this means you can specify a list of functions rather than just one function
(like `p` or `d`). the `[2addr]` means this function is compatible with up to 2
addresses (meaning it can be used with 0, 1, or 2 addresses). function lists are
expressed within `{ }`, with each function specified on its own line.

so back to our last example:

```bash
sed -n '1,6 {
  /^$/ d
  p
}' moby_dick.txt
```

in this function list, we provide 2 functions, the `d` delete function (which
itself only operates on lines selected by the address `/^$/` e.g. empty lines).
the man page tells us that the `d` function _"deletes the pattern space and
starts the next cycle."_. therefore if there was a blank line, the function list
is cut short by the `d` function. if it encounters a _non_-empty line, the
address provided for `d` would not select the line and therefore be skipped. the
next function, `p` would run, meaning we only print non empty lines.

this could have been done more simply using a regex address that selected
non-empty lines and printed them, but then we wouldn't have gotten to see how
function lists work. actually you know what, why not, lets see how we would do
that.

```bash
sed -n '1,6 {
  /./ p
}' moby_dick.txt
```

```
call me ishmael. some years ago—never mind how long precisely—having little or no money in my purse, and nothing particular to interest me on shore, i thought i would sail about a little and see the watery part of the world. it is a way i have of driving off the spleen and regulating the circulation. whenever i find myself growing grim about the mouth; whenever it is a damp, drizzly november in my soul; whenever i find myself involuntarily pausing before coffin warehouses, and bringing up the rear of every funeral i meet; and especially whenever my hypos get such an upper hand of me, that it requires a strong moral principle to prevent me from deliberately stepping into the street, and methodically knocking people’s hats off—then, i account it high time to get to sea as soon as i can. this is my substitute for pistol and ball. with a philosophical flourish cato throws himself upon his sword; i quietly take to the ship. there is nothing surprising in this. if they but knew it, almost all men in their degree, some time or other, cherish very nearly the same feelings towards the ocean with me.
there now is your insular city of the manhattoes, belted round by wharves as indian isles by coral reefs—commerce surrounds it with her surf. right and left, the streets take you waterward. its extreme downtown is the battery, where that noble mole is washed by waves, and cooled by breezes, which a few hours previous were out of sight of land. look at the crowds of water-gazers there.
circumambulate the city of a dreamy sabbath afternoon. go from corlears hook to coenties slip, and from thence, by whitehall, northward. what do you see?—posted like silent sentinels all around the town, stand thousands upon thousands of mortal men fixed in ocean reveries. some leaning against the spiles; some seated upon the pier-heads; some looking over the bulwarks of ships from china; some high aloft in the rigging, as if striving to get a still better seaward peep. but these are all landsmen; of week days pent up in lath and plaster—tied to counters, nailed to benches, clinched to desks. how then is this? are the green fields gone? what do they here?
```

because we needed to perform a regex match on lines within an address range, we
still needed the function list, even though there was only one function in it.
this just prints any line that has any character (newlines are excluded from
`.`) on it.

this could be expressed in yet another way by using address negation.

```bash
sed -n '1,6 {
  /^$/ !p
}' moby_dick.txt
```

```
call me ishmael. some years ago—never mind how long precisely—having little or no money in my purse, and nothing particular to interest me on shore, i thought i would sail about a little and see the watery part of the world. it is a way i have of driving off the spleen and regulating the circulation. whenever i find myself growing grim about the mouth; whenever it is a damp, drizzly november in my soul; whenever i find myself involuntarily pausing before coffin warehouses, and bringing up the rear of every funeral i meet; and especially whenever my hypos get such an upper hand of me, that it requires a strong moral principle to prevent me from deliberately stepping into the street, and methodically knocking people’s hats off—then, i account it high time to get to sea as soon as i can. this is my substitute for pistol and ball. with a philosophical flourish cato throws himself upon his sword; i quietly take to the ship. there is nothing surprising in this. if they but knew it, almost all men in their degree, some time or other, cherish very nearly the same feelings towards the ocean with me.
there now is your insular city of the manhattoes, belted round by wharves as indian isles by coral reefs—commerce surrounds it with her surf. right and left, the streets take you waterward. its extreme downtown is the battery, where that noble mole is washed by waves, and cooled by breezes, which a few hours previous were out of sight of land. look at the crowds of water-gazers there.
circumambulate the city of a dreamy sabbath afternoon. go from corlears hook to coenties slip, and from thence, by whitehall, northward. what do you see?—posted like silent sentinels all around the town, stand thousands upon thousands of mortal men fixed in ocean reveries. some leaning against the spiles; some seated upon the pier-heads; some looking over the bulwarks of ships from china; some high aloft in the rigging, as if striving to get a still better seaward peep. but these are all landsmen; of week days pent up in lath and plaster—tied to counters, nailed to benches, clinched to desks. how then is this? are the green fields gone? what do they here?
```

the effect of the `!` is to only run the given function when the line is **not**
selected by the address.

# the hold space

so there's actually another buffer that `sed` can hold data in while it's
executing. in addition to the pattern space, there's also the _hold space_. when
`sed` is running, it wipes the _pattern space_ clean after each cycle, meaning
on each iteration the pattern space only contains the current input line (by
default).

the _hold space_ does not follow this rule. when you put some text into the hold
space, it stays there until you explicitly clear it or add to it or move it to
the pattern space.

the next handful of examples will cover just why exactly the hold space can be
useful.

## replicating grep --context

the `--context` flag in grep allows you to print the adjacent `num` lines before
and after each match. keeping a memory of lines before seeing a matching line is
exactly the sort of thing `sed`\'s hold pattern can help us with.

```bash
sed -e '/^$/ d' moby_dick.txt | sed -n '
/ship/ !{
  x
  d
}
/ship/ {
  x
  p
  x
  p
  n
  p
  a\
  ---
  x
}'
```

```
call me ishmael. some years ago—never mind how long precisely—having little or no money in my purse, and nothing particular to interest me on shore, i thought i would sail about a little and see the watery part of the world. it is a way i have of driving off the spleen and regulating the circulation. whenever i find myself growing grim about the mouth; whenever it is a damp, drizzly november in my soul; whenever i find myself involuntarily pausing before coffin warehouses, and bringing up the rear of every funeral i meet; and especially whenever my hypos get such an upper hand of me, that it requires a strong moral principle to prevent me from deliberately stepping into the street, and methodically knocking people’s hats off—then, i account it high time to get to sea as soon as i can. this is my substitute for pistol and ball. with a philosophical flourish cato throws himself upon his sword; i quietly take to the ship. there is nothing surprising in this. if they but knew it, almost all men in their degree, some time or other, cherish very nearly the same feelings towards the ocean with me.
there now is your insular city of the manhattoes, belted round by wharves as indian isles by coral reefs—commerce surrounds it with her surf. right and left, the streets take you waterward. its extreme downtown is the battery, where that noble mole is washed by waves, and cooled by breezes, which a few hours previous were out of sight of land. look at the crowds of water-gazers there.
---
there now is your insular city of the manhattoes, belted round by wharves as indian isles by coral reefs—commerce surrounds it with her surf. right and left, the streets take you waterward. its extreme downtown is the battery, where that noble mole is washed by waves, and cooled by breezes, which a few hours previous were out of sight of land. look at the crowds of water-gazers there.
circumambulate the city of a dreamy sabbath afternoon. go from corlears hook to coenties slip, and from thence, by whitehall, northward. what do you see?—posted like silent sentinels all around the town, stand thousands upon thousands of mortal men fixed in ocean reveries. some leaning against the spiles; some seated upon the pier-heads; some looking over the bulwarks of ships from china; some high aloft in the rigging, as if striving to get a still better seaward peep. but these are all landsmen; of week days pent up in lath and plaster—tied to counters, nailed to benches, clinched to desks. how then is this? are the green fields gone? what do they here?
but look! here come more crowds, pacing straight for the water, and seemingly bound for a dive. strange! nothing will content them but the extremest limit of the land; loitering under the shady lee of yonder warehouses will not suffice. no. they must get just as nigh the water as they possibly can without falling in. and there they stand—miles of them—leagues. inlanders all, they come from lanes and alleys, streets and avenues—north, east, south, and west. yet here they all unite. tell me, does the magnetic virtue of the needles of the compasses of all those ships attract them thither?
---
once more. say you are in the country; in some high land of lakes. take almost any path you please, and ten to one it carries you down in a dale, and leaves you there by a pool in the stream. there is magic in it. let the most absent-minded of men be plunged in his deepest reveries—stand that man on his legs, set his feet a-going, and he will infallibly lead you to water, if water there be in all that region. should you ever be athirst in the great american desert, try this experiment, if your caravan happen to be supplied with a metaphysical professor. yes, as every one knows, meditation and water are wedded for ever.
but here is an artist. he desires to paint you the dreamiest, shadiest, quietest, most enchanting bit of romantic landscape in all the valley of the saco. what is the chief element he employs? there stand his trees, each with a hollow trunk, as if a hermit and a crucifix were within; and here sleeps his meadow, and there sleep his cattle; and up from yonder cottage goes a sleepy smoke. deep into distant woodlands winds a mazy way, reaching to overlapping spurs of mountains bathed in their hill-side blue. but though the picture lies thus tranced, and though this pine-tree shakes down its sighs like leaves upon this shepherd’s head, yet all were vain, unless the shepherd’s eye were fixed upon the magic stream before him. go visit the prairies in june, when for scores on scores of miles you wade knee-deep among tiger-lilies—what is the one charm wanting?—water—there is not a drop of water there! were niagara but a cataract of sand, would you travel your thousand miles to see it? why did the poor poet of tennessee, upon suddenly receiving two handfuls of silver, deliberate whether to buy him a coat, which he sadly needed, or invest his money in a pedestrian trip to rockaway beach? why is almost every robust healthy boy with a robust healthy soul in him, at some time or other crazy to go to sea? why upon your first voyage as a passenger, did you yourself feel such a mystical vibration, when first told that you and your ship were now out of sight of land? why did the old persians hold the sea holy? why did the greeks give it a separate deity, and own brother of jove? surely all this is not without meaning. and still deeper the meaning of that story of narcissus, who because he could not grasp the tormenting, mild image he saw in the fountain, plunged into it and was drowned. but that same image, we ourselves see in all rivers and oceans. it is the image of the ungraspable phantom of life; and this is the key to it all.
now, when i say that i am in the habit of going to sea whenever i begin to grow hazy about the eyes, and begin to be over conscious of my lungs, i do not mean to have it inferred that i ever go to sea as a passenger. for to go as a passenger you must needs have a purse, and a purse is but a rag unless you have something in it. besides, passengers get sea-sick—grow quarrelsome—don’t sleep of nights—do not enjoy themselves much, as a general thing;—no, i never go as a passenger; nor, though i am something of a salt, do i ever go to sea as a commodore, or a captain, or a cook. i abandon the glory and distinction of such offices to those who like them. for my part, i abominate all honorable respectable toils, trials, and tribulations of every kind whatsoever. it is quite as much as i can do to take care of myself, without taking care of ships, barques, brigs, schooners, and what not. and as for going as cook,—though i confess there is considerable glory in that, a cook being a sort of officer on ship-board—yet, somehow, i never fancied broiling fowls;—though once broiled, judiciously buttered, and judgmatically salted and peppered, there is no one who will speak more respectfully, not to say reverentially, of a broiled fowl than i will. it is out of the idolatrous dotings of the old egyptians upon broiled ibis and roasted river horse, that you see the mummies of those creatures in their huge bake-houses the pyramids.
---
```

there's a lot going on here. but it's mostly a culmination of everything that
we\'re covered so far so i think we can work through it. first we run the file
contents through a simple `sed` invocation that just deletes all the empty
lines. from there, the edited output is piped into our main `sed` script.

it's composed of two commands: both with a regex address looking for the string,
"ship". in the first case, we provide a function list preceded by `!` meaning
the following functions will be executed on any line that **doesn't** match the
regex `ship`. let's take a deeper look at those two functions.

```
/ship/ !{
  x
  d
}
```

the `d` you should already know, but the `x` is new. our trusty [man](man:sed)
page tells us the `x` function _"swaps the contents of the pattern and hold
spaces."_ so the two functions together with the negated function list and the
regex address say:

on any line that doesn't contain the string "ship", place that line into the
hold space, taking the current contents of the hold space and place them into
the pattern space. then delete the pattern space. effectively, this saves each
line into the hold space just in case the next line **does** match our regex.
because we need to keep a memory of the prior line to display the context around
the match, this will always have the line of the previous cycle in memory.

next let's tackle the bigger section:

```
/ship/ {
  x
  p
  x
  p
  n
  p
  a\
---
  x
}
```

the first section ran for all lines that didn't match our search pattern, and
this is the counterpart. this runs the list of functions over each line that
**does** match our search term.

`x` and then `p` will swap the hold space with the pattern space and then print
it. because of the first section, we know that the hold space will always
contain the contents of the previous cycle. so `x` and `p` together print the
previous line. the next `x` and `p` swap the hold and pattern spaces back,
meaning the current line is back in the pattern space which is then printed.

the `n` is a new function as well. you can think of it as hitting the down arrow
in your text editor. sed wipes the current pattern space, and pulls the next
line into the pattern space. so with `n` and `p`, we print the _next_ line of
input.

the last function we still have to introduce is `a`. `a` writes the given text
_after_ proccessing the current line. the man page has this to say about the `a`
function: _"write `text` to standard output immediately before each attempt to
read a line of input..."_ but i find that description really confusing since it
seems to indicate the text should appear before other things. how it clicked for
me was thinking about it as an append action like in vim. `a` for after, `i` for
before.[^3] for any of the functions that take some `text`, the function must
have a back slash and a newline immediately after it, followed by the text you
wish to add, followed by the newline.[^4]

in this instance:

```
   a\
---
```

means that after we have printed our three lines (before, current, and next), we
append `---` as a marker to make it clear where the current triplet stops and
the next one begins.

lastly, because we've pulled in the next line within our script with the `n`
function, we need to add it to the hold space in order to continue the logic of
adding each "previous" line to the hold space.

here's the script again in all its glory:

```bash
sed -e '/^$/ d' moby_dick.txt | sed -n '
/ship/ !{
  x
  d
}
/ship/ {
  x
  p
  x
  p
  n
  p
  a\
  ---
  x
}'
```

```
call me ishmael. some years ago—never mind how long precisely—having little or no money in my purse, and nothing particular to interest me on shore, i thought i would sail about a little and see the watery part of the world. it is a way i have of driving off the spleen and regulating the circulation. whenever i find myself growing grim about the mouth; whenever it is a damp, drizzly november in my soul; whenever i find myself involuntarily pausing before coffin warehouses, and bringing up the rear of every funeral i meet; and especially whenever my hypos get such an upper hand of me, that it requires a strong moral principle to prevent me from deliberately stepping into the street, and methodically knocking people’s hats off—then, i account it high time to get to sea as soon as i can. this is my substitute for pistol and ball. with a philosophical flourish cato throws himself upon his sword; i quietly take to the ship. there is nothing surprising in this. if they but knew it, almost all men in their degree, some time or other, cherish very nearly the same feelings towards the ocean with me.
there now is your insular city of the manhattoes, belted round by wharves as indian isles by coral reefs—commerce surrounds it with her surf. right and left, the streets take you waterward. its extreme downtown is the battery, where that noble mole is washed by waves, and cooled by breezes, which a few hours previous were out of sight of land. look at the crowds of water-gazers there.
---
there now is your insular city of the manhattoes, belted round by wharves as indian isles by coral reefs—commerce surrounds it with her surf. right and left, the streets take you waterward. its extreme downtown is the battery, where that noble mole is washed by waves, and cooled by breezes, which a few hours previous were out of sight of land. look at the crowds of water-gazers there.
circumambulate the city of a dreamy sabbath afternoon. go from corlears hook to coenties slip, and from thence, by whitehall, northward. what do you see?—posted like silent sentinels all around the town, stand thousands upon thousands of mortal men fixed in ocean reveries. some leaning against the spiles; some seated upon the pier-heads; some looking over the bulwarks of ships from china; some high aloft in the rigging, as if striving to get a still better seaward peep. but these are all landsmen; of week days pent up in lath and plaster—tied to counters, nailed to benches, clinched to desks. how then is this? are the green fields gone? what do they here?
but look! here come more crowds, pacing straight for the water, and seemingly bound for a dive. strange! nothing will content them but the extremest limit of the land; loitering under the shady lee of yonder warehouses will not suffice. no. they must get just as nigh the water as they possibly can without falling in. and there they stand—miles of them—leagues. inlanders all, they come from lanes and alleys, streets and avenues—north, east, south, and west. yet here they all unite. tell me, does the magnetic virtue of the needles of the compasses of all those ships attract them thither?
---
once more. say you are in the country; in some high land of lakes. take almost any path you please, and ten to one it carries you down in a dale, and leaves you there by a pool in the stream. there is magic in it. let the most absent-minded of men be plunged in his deepest reveries—stand that man on his legs, set his feet a-going, and he will infallibly lead you to water, if water there be in all that region. should you ever be athirst in the great american desert, try this experiment, if your caravan happen to be supplied with a metaphysical professor. yes, as every one knows, meditation and water are wedded for ever.
but here is an artist. he desires to paint you the dreamiest, shadiest, quietest, most enchanting bit of romantic landscape in all the valley of the saco. what is the chief element he employs? there stand his trees, each with a hollow trunk, as if a hermit and a crucifix were within; and here sleeps his meadow, and there sleep his cattle; and up from yonder cottage goes a sleepy smoke. deep into distant woodlands winds a mazy way, reaching to overlapping spurs of mountains bathed in their hill-side blue. but though the picture lies thus tranced, and though this pine-tree shakes down its sighs like leaves upon this shepherd’s head, yet all were vain, unless the shepherd’s eye were fixed upon the magic stream before him. go visit the prairies in june, when for scores on scores of miles you wade knee-deep among tiger-lilies—what is the one charm wanting?—water—there is not a drop of water there! were niagara but a cataract of sand, would you travel your thousand miles to see it? why did the poor poet of tennessee, upon suddenly receiving two handfuls of silver, deliberate whether to buy him a coat, which he sadly needed, or invest his money in a pedestrian trip to rockaway beach? why is almost every robust healthy boy with a robust healthy soul in him, at some time or other crazy to go to sea? why upon your first voyage as a passenger, did you yourself feel such a mystical vibration, when first told that you and your ship were now out of sight of land? why did the old persians hold the sea holy? why did the greeks give it a separate deity, and own brother of jove? surely all this is not without meaning. and still deeper the meaning of that story of narcissus, who because he could not grasp the tormenting, mild image he saw in the fountain, plunged into it and was drowned. but that same image, we ourselves see in all rivers and oceans. it is the image of the ungraspable phantom of life; and this is the key to it all.
now, when i say that i am in the habit of going to sea whenever i begin to grow hazy about the eyes, and begin to be over conscious of my lungs, i do not mean to have it inferred that i ever go to sea as a passenger. for to go as a passenger you must needs have a purse, and a purse is but a rag unless you have something in it. besides, passengers get sea-sick—grow quarrelsome—don’t sleep of nights—do not enjoy themselves much, as a general thing;—no, i never go as a passenger; nor, though i am something of a salt, do i ever go to sea as a commodore, or a captain, or a cook. i abandon the glory and distinction of such offices to those who like them. for my part, i abominate all honorable respectable toils, trials, and tribulations of every kind whatsoever. it is quite as much as i can do to take care of myself, without taking care of ships, barques, brigs, schooners, and what not. and as for going as cook,—though i confess there is considerable glory in that, a cook being a sort of officer on ship-board—yet, somehow, i never fancied broiling fowls;—though once broiled, judiciously buttered, and judgmatically salted and peppered, there is no one who will speak more respectfully, not to say reverentially, of a broiled fowl than i will. it is out of the idolatrous dotings of the old egyptians upon broiled ibis and roasted river horse, that you see the mummies of those creatures in their huge bake-houses the pyramids.
---
```

there are a handful of additional functions to explore, but this article has
already gone on far longer than i anticipated, so feel free to take your
newfound knowledge and experiment with them.

# conclusion

the examples (and a great deal of my understanding of how sed works) has come
from this article: <https://www.grymoire.com/unix/sed.html>. I owe bruce a great
debt for taking the terse man page for `sed` and bringing it alive. I wouldn't
have been able to write this article without his writing.

as for the `sed` tool itself, there was a major mental shift that occurred to me
as I began to understand how it thinks about the world. I pictured it as a
simple line processing tool similar to [grep](/posts/grep) or [tr](man:tr), but
the more I started to see things `sed`\'s way, I started to see its power. It's
not an exaggeration that `sed` is called a stream _editor_.

`sed` equips you with the same sort of editing commands an interactive text
editor provides, but in a stream based way. this yields two primary benefits: 1)
you can use it to edit more than just files. anything that outputs text can be
fed through `sed` using pipelines. and 2) the editing commands you assemble with
sed can be saved in a bash script or even as a sed script itself (check out the
`-f` flag) and repeated whenever you need to wrangle text the same way again in
the future. these scripts can be made executable and saved to your path, adding
more custom tools to your command line toolbelt.

when does it make sense to use `sed` rather than a bash script or lugging python
out for fancier changes? that depends. when do you need to just open up an
interactive editor and poke around? that also depends on what you're looking for
and what you're trying to change. I'm just trying to give you another option,
rather than _needing_ to open up vs code or some other application just for
simple, repetitive edits. when your primary aim with sed is to read in a file,
make some changes and save the results back to the same file, you can use
`sed`'s `-i` flag.[^5]

hopefully this pulls back the curtain on `sed` a bit. I don't think you should
always reach for sed when building a shell pipeline to edit some text, but you
should at least ask yourself, could i just do all of this using `sed`?

# footnotes

[^1]: In this article, the input text is the first chapter of the novel _Moby
    Dick_. accessed from
    [www.gutenberg.org](https://www.gutenberg.org/cache/epub/2701/pg2701-images.html#link2HCH0001)

[^2]: The [sed](https://man7.org/linux/man-pages/man1/sed.1.html) man page is
    helpful here to understand the differences.

[^3]: sed does support the `i` function, which writes the text before the next
    command.

[^4]: thinking about `sed` as a text editor really helps to understand how it
    sees the world. if you\'re famaliar with vim, you know of `d` to delete
    things, `i` to insert before, `a` to insert after, `c` to change things.
    these all have counterparts in `sed`, but all are _line-oriented_. so `d`
    deletes the current line from the input stream, `i` inserts a given line
    before the current line, `a` inserts after, and `c` replaces the current
    line with the given `text`.

[^5]: I recently updated the layout of posts on this blog and needed to update
    the links in all the articles to match the update. once i figured out what
    changes i needed to make, `sed` performed all the heavy lifting for me:

    ```bash
    for file in *.org; do
      sed -e 's#\.\./images#images#' -e 's#terminal_tooling/##' -i '.bak' "$file"
    done
    ```

    The `#` is the separator for my substitution commands. most examples of the
    `s` function use `/`, but it's not strictly necessary. whichever token you
    provide after the `s` becomes the deliniator between each component of the
    substitution.

    This has the effect of going through each article source file and replacing
    each image link with the same path, one directory up. the `-e` allows you to
    specify multiple scripts which `sed` will run serially.

    The `-i '.bak'` tells `sed` to make backups of each file it changes so you
    can recover the pre-changed version in case things went wrong.

    p.s. when setting up a blog which auto executes embedded scripts, don\'t let
    it run the scripts that mess with the articles themselves. things
    get....weird.
