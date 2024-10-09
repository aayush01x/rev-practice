# Tutorial
## main
Looking at the disassembly of the main function:
```assembly
4438 <main>
4438:  3150 9cff      add	#0xff9c, sp
443c:  3f40 a844      mov	#0x44a8 "Enter the password to continue", r15
4440:  b012 5845      call	#0x4558 <puts>
4444:  0f41           mov	sp, r15
4446:  b012 7a44      call	#0x447a <get_password>
444a:  0f41           mov	sp, r15
444c:  b012 8444      call	#0x4484 <check_password>
4450:  0f93           tst	r15
4452:  0520           jnz	$+0xc <main+0x26>
4454:  3f40 c744      mov	#0x44c7 "Invalid password; try again.", r15
4458:  b012 5845      call	#0x4558 <puts>
445c:  063c           jmp	$+0xe <main+0x32>
445e:  3f40 e444      mov	#0x44e4 "Access Granted!", r15
4462:  b012 5845      call	#0x4558 <puts>
4466:  b012 9c44      call	#0x449c <unlock_door>
446a:  0f43           clr	r15
446c:  3150 6400      add	#0x64, sp
```

First we adjust the stack pointer, after that the program puts  `Enter the password to continue`.

Then it moves stack pointer into `r15`, so `r15` is passed as parameter to the `get_password` function.

Then `check_password` function is called with `r15` as parameter. After this, it is checked if `r15` is `0` using `tst`.

If not, then `jnz` instruction is called which makes jump to instruction `<main+0x26>` which is `445e`, this puts `Access Granted!` and calls the `unlock_door` function.

Otherwise, there is no jump, and the program puts `"Invalid password; try again."` and jumps to `446a`.

So, we want that `r15` **is not** zero after `check_password` function is called.

## check_password

Disassembly:
```assembly
4484 <check_password>
4484:  6e4f           mov.b	@r15, r14
4486:  1f53           inc	r15
4488:  1c53           inc	r12
448a:  0e93           tst	r14
448c:  fb23           jnz	$-0x8 <check_password+0x0>
448e:  3c90 0900      cmp	#0x9, r12
4492:  0224           jz	$+0x6 <check_password+0x14>
4494:  0f43           clr	r15
4496:  3041           ret
4498:  1f43           mov	#0x1, r15
449a:  3041           ret
```
`r15` will store location of the first character of our input password. `mov.b	@r15, r14` moves first character into `r14` as `@r15` is dereferencing address at `r15`.

Then `r15` is incremented and `r12` is also incremented. `r12` is originally 0. Then program tests if `r14` is zero, if not then it jumps to start of the `check_password` function again. The only difference this time is that `r15` and `r12` have been incremented in comparison to the last time.

So, When will we reach instruction `448e` ? It will be possible when the `r14` register contains 0 value or the character which `r15` points to is the **null byte** at the end of the string.

So, basically `check_password` function iterates in string, stores length of string in `r12`. So, when we reach `448e` instruction, the value of `r12` will be `length_of_input_password + 1` as it would be have incremented while `r15` point to null byte too.

Program then checks if `r12` is equal to `0x9` and jumps to `4498` if true. We want this to be True, because then `0x1` is moved into `r15`, and hence `r15` is not zero. Otherwise, the program clears value of `r15` and returns.

So, we just want `length_of_input_password + 1 = 9`.

Hence, **length_of_input_password = 8**. Any password which satisfies this, should work.

So, we enter the password `abcdefgh`, and it successfully works!

![alt text](../imgs/mc-tut/image.png)

![alt text](../imgs/mc-tut/image-1.png)
