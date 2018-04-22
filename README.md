# netlogo_dynamics_modeling
extensions [matrix]
globals [i j] ;; this is where you create global (static) variables

;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;
patches-own[
  Aij1
  Aij2
  Aij3
  Aij4
  Eij
  D-ij
  Dij1
  Di2j
  Di0j
  Dij2
  Dij0
]

turtles-own[
  P ;;probability
  obstacle   ;;other people
  xi0
  xi1
  xi2
  xi3
  xi4
  occupy0 ;; self occupation
  occupy1
  occupy2
  occupy3
  occupy4 ;; = 0 or 1 if pedestrian occupy the cell
  time-to-cross
  xprev
  yprev
  direction0
  Eij-d
  eRight
  eDown
  eLeft
  eUp
  dRight
  dDown
  dLeft
  dUp
  dStay
]

to setup
  clear-all
  setup-patches
  check-patches
  reset-ticks
  ask turtles [
  set eRight 0
  set eDown 0
  set eLeft 0
  set eUp 0]
end


to setup-patches
  ask patches [
    ifelse pycor mod 2 = 0 [set pcolor 8] [set pcolor 9]
  ]

  ask n-of n-blue-agents patches [
    sprout 1 [
      set shape "person"
      set size 1.2
      set heading 90
      set color blue
      set xcor (random 6)
      set direction0 1
    ]
  ]

  ask n-of n-red-agents patches [
    sprout 1 [
      set shape "person"
      set size 1.2
      set heading 90
      set color red
      set xcor (79 - random 6)
      set direction0 3
    ]
  ]

end



to check-patches
  ask patches [
    while [count turtles-here >= 2]
    [ if any? turtles-here with [color = red]
      [ask one-of turtles-here with [color = red] [set xcor (xcor - 1)]]

      if any? turtles-here with [color = blue]
        [ask one-of turtles-here with [color = blue]   [set xcor (xcor + 1)] ]
    ]
    if count turtles-here >= 2
      [ set pcolor 1 ]
  ]

end




to go
  get-parameter
  calc-AFF
  move-turtles

  tick
  if count turtles = 0 [stop]
  if ticks > 5000 [stop]
end

;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;
to calc-AFF
  ask patches[
    set Aij1 0
    set Aij2 0
    set Aij3 0
    set Aij4 0]

   ask turtles [
    get-AFF-parameter xcor ycor direction0 dA
  ]


end






to move-turtles

  ask turtles[
    set xprev xcor
    set yprev ycor

    ;let val matrix:get mA yprev xprev
    ;set mB matrix:set-and-report mB yprev xprev (val + 1)

    let sRight get-static-FF color xcor ycor 1
    let sDown get-static-FF color xcor ycor 2
    let sLeft get-static-FF color xcor ycor 3
    let sUp get-static-FF color xcor ycor 4

    set eRight set-Eij xcor ycor 1
    set eDown set-Eij xcor ycor 2
    set eLeft set-Eij xcor ycor 3
    set eUp set-Eij xcor ycor 4

    set dRight get-dynamic-FF xcor ycor 1
    set dDown get-dynamic-FF xcor ycor 4
    set dLeft get-dynamic-FF xcor ycor 3
    set dUp get-dynamic-FF xcor ycor 2
    set dStay get-dynamic-FF xcor ycor 0

    let p1 (get-transition-prob xi1 eRight sRight dRight occupy1) ;; probability of moving right
    let p2 (get-transition-prob xi2 eDown sDown dDown occupy2) ;; probability of moving down
    let p3 (get-transition-prob xi3 eLeft sLeft dLeft occupy3) ;; probability of moving left
    let p4 (get-transition-prob xi4 eUp sUp dUp occupy4) ;; probability of moving up
    let p5 (get-transition-prob xi0 0 sUp dStay occupy0) ;; probability of staying still

    let direction (get-direction p1 p2 p3 p4 p5)
    set direction0 (direction / 90 + 1)


    ifelse direction = 360
    [ forward 0 ] ;; stay still
    [ right direction
      forward 1
      set heading 90 ]




  ];; end ask turtles

  ;; resolve conflicts
  resolve-conflicts




  ;; check if turtles have crossed the corridor
  ask turtles[
     ask turtles with [ color = blue ]
      [ if xcor = 80
        [ set time-to-cross ticks
          die ] ]
      ask turtles with [ color = red ]
      [ if xcor = 0
        [ set time-to-cross ticks
          die ] ]
  ]
end
;;;;;;;;;;;;;;;;;;;;;;;;;;

to resolve-conflicts
  ask patches [
    if count turtles-here >= 2
    [ ask one-of turtles-here
      [ set xcor xprev
        set ycor yprev ] ]

    if count turtles-here >= 2
    [ ask one-of turtles-here
      [ set xcor xprev
        set ycor yprev ] ]

    if count turtles-here >= 2
    [ ask one-of turtles-here
      [ set xcor xprev
        set ycor yprev ] ]

    if count turtles-here >= 2
    [ ask one-of turtles-here
      [ set xcor xprev
        set ycor yprev ] ]

    if count turtles-here >= 2
    [ ask one-of turtles-here
      [ set xcor xprev
        set ycor yprev ] ]


 ifelse count turtles-here >= 2[

    ]
    [ifelse pycor mod 2 = 0 [set pcolor 8] [set pcolor 9] ]

  ]
end






to-report get-static-FF [col xpos ypos direction]
  let sff 0
  ifelse col = 15 ;; red
  [
    if direction = 1 ;; right
    [set sff (xpos + 1)]
    if direction = 2 ;; down
    [set sff xpos ]
    if direction = 3 ;; left
    [set sff (xpos - 1)]
    if direction = 4 ;; up
    [set sff xpos ]
  ]
  [ if direction = 1 ;; right
    [set sff (80 - xpos - 1)]
    if direction = 2 ;; down
    [set sff (80 - xpos) ]
    if direction = 3 ;; left
    [set sff (80 - xpos + 1)]
    if direction = 4 ;; up
    [set sff (80 - xpos) ]  ]
  report sff
end



to-report get-dynamic-FF [xpos ypos direction]
  ;let dff matrix:get mA ypos xpos

    if (xprev != xcor) or (yprev != ycor) [

      ask patch-at xpos ypos [set D-ij D-ij + 1]
    ]

      if patch-at xpos ypos = nobody [
        if direction = 0 [
          ask patch-at (xpos + 1) ypos [set Di2j D-ij]
          ask patch-at (xpos - 1) ypos [set Di0j D-ij]
          ask patch-at xpos (ypos + 1) [set Dij2 D-ij]
          ask patch-at xpos (ypos - 1) [set Dij0 D-ij]

          set D-ij (1 - Phi) * (1 - (1 / dA)) * D-ij + (Phi * (1 - (1 / dA)) * (Di2j + Di0j + Dij2 + Dij0) ) / 4

          if D-ij < 0 [set D-ij 0]
        ]

        if direction = 1 [
          ask patch-at (xpos + 1) ypos [set Dij1 D-ij]
          ask patch-at (xpos + 2) ypos [set Di2j D-ij]
          ask patch-at xpos ypos [set Di0j D-ij]
          ask patch-at (xpos + 1) (ypos + 1) [set Dij2 D-ij]
          ask patch-at (xpos + 1) (ypos - 1) [set Dij0 D-ij]

          set D-ij (1 - Phi) * (1 - (1 / dA)) * Dij1 + (Phi * (1 - (1 / dA)) * (Di2j + Di0j + Dij2 + Dij0) ) / 4

          if D-ij < 0 [set D-ij 0]
        ]

        if direction = 2 [
          ask patch-at xpos (ypos + 1) [set Dij1 D-ij]
          ask patch-at (xpos + 1) (ypos + 1) [set Di2j D-ij]
          ask patch-at (xpos - 1) (ypos + 1) [set Di0j D-ij]
          ask patch-at (xpos) (ypos + 2) [set Dij2 D-ij]
          ask patch-at xpos ypos [set Dij0 D-ij]

          set D-ij (1 - Phi) * (1 - (1 / dA)) * Dij1 + (Phi * (1 - (1 / dA)) * (Di2j + Di0j + Dij2 + Dij0) ) / 4

          if D-ij < 0 [set D-ij 0]
        ]


        if direction = 3 [
          ask patch-at (xpos - 1) ypos [set Dij1 D-ij]
          ask patch-at xpos ypos [set Di2j D-ij]
          ask patch-at (xpos - 2) ypos [set Di0j D-ij]
          ask patch-at (xpos - 1) (ypos + 1) [set Dij2 D-ij]
          ask patch-at (xpos - 1) (ypos - 1) [set Dij0 D-ij]

          set D-ij (1 - Phi) * (1 - (1 / dA)) * Dij1 + (Phi * (1 - (1 / dA)) * (Di2j + Di0j + Dij2 + Dij0) ) / 4

          if D-ij < 0 [set D-ij 0]
        ]

        if direction = 4 [
          ask patch-at xpos (ypos - 1) [set Dij1 D-ij]
          ask patch-at (xpos + 1) (ypos - 1) [set Di2j D-ij]
          ask patch-at (xpos - 1) (ypos - 1) [set Di0j D-ij]
          ask patch-at xpos ypos [set Dij2 D-ij]
          ask patch-at xpos (ypos - 2) [set Dij0 D-ij]

          set D-ij (1 - Phi) * (1 - (1 / dA)) * Dij1 + (Phi * (1 - (1 / dA)) * (Di2j + Di0j + Dij2 + Dij0) ) / 4

          if D-ij < 0 [set D-ij 0]
        ]
     ]

  report D-ij

end



to get-AFF-parameter [xpos ypos direction n]
  ;show direction
  let h 1
  while[h <= n][
    if (direction = 1)[
    if patch-at (xpos + h) ypos != nobody[
        ask patch-at (xpos + h) ypos [set Aij1 Aij1 + 1]]
    ]
    if (direction = 2)[
    if patch-at xpos (ypos - h) != nobody[
      ask patch-at xpos (ypos - h) [set Aij2 Aij2 + 1]]
  ]
    if (direction = 3)[
      if patch-at (xpos - h) ypos != nobody[
        ask patch-at (xpos - h) ypos [set Aij3 Aij3 + 1]]
  ]
    if (direction = 4)[
      if patch-at xpos (ypos + h) != nobody[
        ask patch-at xpos (ypos + h) [set Aij4 Aij4 + 1]]
  ]
    set h h + 1
  ]
end

to-report set-Eij [xpos ypos direction]
  set Eij 0
    if (direction = 1)[
    if patch-at (xpos + 1) ypos != nobody[
      ask patch-at (xpos + 1) ypos [set Eij Aij2 + Aij3 + Aij4]]
  ]
    if (direction = 2)[
    if patch-at xpos (ypos - 1) != nobody[
      ask patch-at xpos (ypos - 1) [set Eij Aij1 + Aij3 + Aij4]]
  ]
    if (direction = 3)[
    if patch-at (xpos - 1) ypos != nobody[
      ask patch-at (xpos - 1) ypos [set Eij Aij1 + Aij2 + Aij4]]
  ]
    if (direction = 4)[
    if patch-at xpos (ypos + 1) != nobody[
    ask patch-at xpos (ypos + 1) [set Eij Aij1 + Aij2 + Aij3]]
  ]
  report Eij
end


to get-parameter

    ask turtles[
    set occupy0 0
    set xi0 1
    set xi1 1
    set xi2 1
    set xi3 1
    set xi4 1

    ifelse nobody = patch-right-and-ahead 0 1
      [ set xi1 0 ]
      [ ifelse any? turtles-on patch-right-and-ahead 0 1
          [ set occupy1 1 ]
          [ set occupy1 0 ] ]
    ifelse nobody = patch-right-and-ahead 90 1
      [ set xi2 0 ]
      [ ifelse any? turtles-on patch-right-and-ahead 90 1
          [ set occupy2 1 ]
          [ set occupy2 0 ] ]
    ifelse nobody = patch-right-and-ahead 180 1
      [ set xi3 0 ]
      [ ifelse any? turtles-on patch-right-and-ahead 180 1
            [ set occupy3 1 ]
            [ set occupy3 0 ] ]
    ifelse nobody = patch-right-and-ahead 270 1
      [ set xi4 0 ]
      [ ifelse any? turtles-on patch-right-and-ahead 270 1
            [ set occupy4 1 ]
            [ set occupy4 0 ] ]

  ]
end




to-report get-transition-prob [xi Eij-m Sij Dij nij]
   report xi * e ^ ( (- kA) * Eij-m) * e ^ ( (- kS) * Sij)  * e ^ ( kD * Dij) * (1 - (Phi * nij))
end



;; determine which direction to move
to-report get-direction [p1 p2 p3 p4 p5]
  let N (p1 + p2 + p3 + p4 + p5)
  let R random-float N
  let direction -1
  if R < p1
      [set direction 0] ;; move right
  if (R >= p1) and (R < (p1 + p2) )
      [set direction 1] ;; move down
  if (R >= (p1 + p2) ) and ( R < (p1 + p2 + p3) )
      [set direction 2 ] ;; move left
  if (R >= (p1 + p2 + p3 ) ) and (R < (p1 + p2 + p3 + p4) )
      [set direction 3] ;; move up
  if (R >= (p1 + p2 + p3 + p4))
      [set direction 4] ;; do not move (stay still)
  report (direction * 90)

end
