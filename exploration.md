# Poking around the Eurisko code

The intro to the program is the `Eurisko` function on L652, which takes a verbosity setting and whether it should run forever (I think?). This then runs an `InitializeEurisko` function to set up a bunch of stuff (it is so cursed how much global variables are used) and then calls the `START` function, which runs the actual program.

Also, since it's written in an old dialect of lisp (Interlisp), it seems that closing paranthesis is optional sometimes??

## InitializeEurisko
L1108

* `DoIt` seems to be the equivalent of `-y` in other programs to skip the question whether or not to initialize.
* Sets some stuff: (`Conjectures`, `UnusedSlots`, `UsedSlots`, `Units`, and `Slots` which is the combination of `UnusedSlots` and `UsedSlots`)
```CL
(SETQ Agenda NIL)
        (SETQ Conjectures NIL)
        (SETQ UnusedSlots NIL)
        (SETQ UsedSlots NIL)
        [MAPC Units (FUNCTION (LAMBDA (U)
                                (MAPC (PROPNAMES U)
                                      (FUNCTION (LAMBDA (SL)
                                                  (OR (MEMB SL UsedSlots)
                                                      (MEMB SL SYSPROPS)
                                                      (PROGN (SETQ UsedSlots (CONS SL UsedSlots))
                                                             (DefineSlot SL]
        [MAPC Units (FUNCTION (LAMBDA (u)
                                (AND (MEMB 'Slot (IsA u))
                                     (NOT (MEMB u UsedSlots))
                                     (SETQ UnusedSlots (CONS u UnusedSlots))
                                     (DefineSlot u]
        (SETQ UsedSlots (SORT UsedSlots))
        (SETQ UnusedSlots (SORT UnusedSlots))
        (MAPC UnusedSlots 'DefineSlot)
```
* Some shit happens here (TODO analyze):
```CL
[AND (SETQ NUnitSlots (SUBSET Slots 'NUnitp))
             (YesNo NIL (CONCAT (LENGTH NUnitSlots)
                               " slots aren't defined as units.  Do that now? "))
             (MAPC (APPEND NUnitSlots)
                   (FUNCTION (LAMBDA (Z)
                               (TERPRI TTY)
                               (PRINT Z TTY)
                               (NU Z 'Abbrev)
                               (SETQ NUnitSlots (DREMOVE Z NUnitSlots]
        (AND NewU (CPRIN1 -1 CRLF "Eliminate the recently synthesized units? ")
             (CPRIN1 20 NewU)
             (YesNo)
             (Map&Print (COPY NewU)
                    'KillUnit))
        (AND (SomeUneliminated)
             (CPRIN1 -1 CRLF "Eliminate the individual values filled in during an earlier run, for slots of units still in existence? "
                    )
             (YesNo)
             (MAPC Units 'InitialElimSlots]
       (T (PRIN1 " OK, just initializing the slot definitions. " TTY)
          (TERPRI TTY)
          [MAPC Units (FUNCTION (LAMBDA (U)
                                  (MAPC (PROPNAMES U)
                                        (FUNCTION (LAMBDA (SL)
                                                    (OR (MEMB SL SYSPROPS)
                                                        (DefineSlot SL]
          (MAPC Units (FUNCTION (LAMBDA (u)
                                  (AND (MEMB 'Slot (IsA u))
                                       (DefineSlot u]
    (CPRIN1 20 CRLF "There are " (LENGTH Units)
           " units, of which "
           (LENGTH SynthU)
           " were synthesized by Eurisko." CRLF)
    (CPRIN1 21 "Of those, " CRLF)
    (ReportOn '(Heuristic MathOp MathObj ReprConcept)
           21)
    (CPRIN1 20 CRLF)
    '!])
```
## START
L2180

* First calls `CycleThruAgenda`
* Defines a PROG which LOOPs with the following condition:

```CL
(COND
             ((SETQ UU (SetDiff Units UnitsFocusedOn)))
             (EternalFlg (CPRIN1 3 CRLF CRLF CRLF 
                  "Have focused on all the units at least once.  Starting another pass through them."
                                CRLF CRLF CRLF)
                    (SETQ UnitsFocusedOn NIL))
             (T (PRIN1 "
Should I continue with another pass? ")
                (OR (YesNo)
                    (RETURN 'EuriskoHalting))
                (SETQ UnitsFocusedOn NIL)))
```


* The core seems to be: (In particular, calling `WorkOnUnit` on the maximum `Worth` Unit)
```CL
(SETQ UnitsFocusedOn (CONS (WorkOnUnit (MAXIMUM UU 'Worth))
                                     UnitsFocusedOn))
```

* Which is followed by (graphics related?)

```CL
(COND
             ((AND (IsAlto)
                   (NULL Agenda))
              (DSPFILL NIL NIL NIL BitAgenda)
              (DSPXPOSITION 2 BitAgenda)
              (DSPYPOSITION 280 BitAgenda)
              (PRIN1 (CONS (LENGTH UU)
                           '(concepts still must be focused on sometime))
                     BitAgenda)
              (BITBLT BitAgenda 0 0 InfoW 406 0))
             (T NIL))             
```

## CycleThruAgenda
L472

* Is also called at the end of `WorkOnUnit`, so probably this is something like "go to the next step"

```CL
(CycleThruAgenda
  [LAMBDA NIL                                                (* edited%: "15-FEB-81 16:25")
    (PROG (task)
      TLOOP
          (COND
             (Agenda (SETQ task (CAR Agenda))
                    (SETQ Agenda (CDR Agenda))
                    (WorkOnTask task)                        (* Note that this might add/change the 
                                                             Agenda)
                    T)
             (T (RETURN NIL)))
          (GO TLOOP])
```

* Core is `WorkOnTask`?
* I assume `T` is the returned Task?

## WorkOnTask
L2894

* Seems to be (one of?) the main functions

```CL
(WorkOnTask
  [LAMBDA (task ArgU TaskResults TimeThen)                   (* edited%: "18-MAY-81 14:33")
    (SETQ AbortTask? NIL)
    (SETQ TaskNum (ADD1 TaskNum))
    (COND
       ((IGREATERP Verbosity 88)
        (TERPRI)
        (PRIN1 "Task ")
        (PRIN1 TaskNum)
        (PRIN1 ": ")
        (PRIN1 "Working on the promising task ")
        (PRIN1 task)
        (TERPRI))
       ((IGREATERP Verbosity 10)
        (CPRIN1 1 CRLF "Task " TaskNum ":  Working on a new promising task:  " (WaxOn task)
               CRLF))
       (T (CPRIN1 0 CRLF "Task " TaskNum CRLF)))
    (SETQ CurPri (ExtractPriority task))
    (SETQ ArgU task)
    (SETQ CurUnit (ExtractUnitName task))
    (SETQ CurSlot (ExtractSlotName task))
    (SETQ CurVal (SETQ OldVal (APPLY* CurSlot CurUnit)))
    (SETQ NewValues NIL)
    (SETQ CurReasons (ExtractReasons task))
    (SETQ CurSup (CurSup task))
    (AND (IsAlto)
         (SnazzyTask)
         (SnazzyAgenda)
         (SnazzyConcept T))
    [OR [EVERY (SubSlots 'IfTaskParts)
               (FUNCTION (LAMBDA (p)
                           (SETQ HeuristicAgenda (Examples 'Heuristic))
                           (PROG (r)
                             HLOOP
                                 (COND
                                    (AbortTask? (RETURN NIL))
                                    ((NULL HeuristicAgenda)
                                     (RETURN T)))
                                 (SETQ r (CAR HeuristicAgenda))
                                 (SETQ HeuristicAgenda (CDR HeuristicAgenda))
                                 (COND
                                    ((NULL (APPLY* p r))
                                     (GO HLOOP))
                                    ((SubsumedBy r)
                                     (GO HLOOP))
                                    ([SELECTQ (APPLY* (APPLY* p r)
                                                     task)
                                         (AbortTask (PUT r 'NAborts (ADD1 (OR (NAborts r)
                                                                              0)))
                                                    (RETURN NIL))
                                         (NIL NIL)
                                         (AND (CPRIN1 66 "	The " p " slot of heuristic " r
                                                     (Abbrev r)
                                                     " applies to the current task. " CRLF)
                                              (OR (AND (IsAlto)
                                                       (SnazzyHeuristic r p))
                                                  T)
                                              (MyTime '(EVERY (SubSlots 'ThenParts)
                                                              'XeqIfItExists)
                                                     'TimeThen)
                                              (OR (AND (IsAlto)
                                                       (SnazzyConcept T))
                                                  T)
                                              (CPRIN1 68 
                                                  "	The Then Parts of the rule have been executed. 
" CRLF)
                                              [SETQ TimRec (OR (OverallRecord r)
                                                               (PUT r 'OverallRecord (CONS 0 0]
                                              (RPLACD TimRec (ADD1 (CDR TimRec)))
                                              (RPLACA TimRec (IPLUS (CAR TimRec)
                                                                    TimeThen]
                                     (GO HLOOP))
                                    (T (GO HLOOP)))
                                 (GO HLOOP]
        (SETQ TaskResults (AddPropL TaskResults 'Termination 'Aborted]
    (CPRIN1 64 " The results of this task were: " TaskResults CRLF)
    (CPRIN1 65 CRLF)
    TaskResults])
```

## WorkOnUnit

* Other main function is seems
```CL
(WorkOnUnit
  [LAMBDA (U TaskResults)                                    (* edited%: "18-MAY-81 17:39")
    (SETQ TaskNum (ADD1 TaskNum))
    (AND (IsAlto)
         (PROGN [SnazzyTask (LIST (Worth U)
                                  U
                                  'any
                                  (LIST '(There are no great tasks on the Agenda now)
                                        (CONS U
                                              '(has the highest Worth of any concept I haven't 
                                                    focused on recently]
                (SnazzyConcept T U)))
    (COND
       ((IGREATERP Verbosity 10)
        (TERPRI)
        (PRIN1 "Task ")
        (PRIN1 TaskNum)
        (PRIN1 ": ")
        (PRIN1 "Focusing on ")
        (PRIN1 U)
        (TERPRI)))
    [MAPC (Examples 'Heuristic)
          (FUNCTION (LAMBDA (H)                              (* try to apply H to unit U)
                      (APPLY* Interp H U]
    (CPRIN1 65 CRLF)
    (AND TaskResults (CPRIN1 64 " The results of this task so far are: " TaskResults CRLF))
    (CPRIN1 65 CRLF)
    (AND (IsAlto)
         (SnazzyHeuristic NIL))
    (CycleThruAgenda)
    U])
```