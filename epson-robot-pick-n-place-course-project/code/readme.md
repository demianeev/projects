Integer Tokens
Integer Blocks
Integer StackPieces
Double TokenHeight
Double BlockHeight
Double Nozzle_Height_Compensation
Double Nozzle_Tray_Height_Compensation
Integer i

Integer TokenID
Integer BlockID

Real timeTask1 'Time elapsed for task1
Real timeTask2 'Time elapsed for task2


Function main

Motor On
Power Low
Speed 15
Accel 15, 15
SpeedS 1500
AccelS 2200, 2200
Tool 1

TokenHeight = 6.0
BlockHeight = 6.0
Nozzle_Height_Compensation = 0.0		'Change to define picking height
Nozzle_Tray_Height_Compensation = 0.0	'Change to define dropping height to the tray

Pallet 1, P14, P17, P16, 2, 3		'Tray pallet


	'Z axis is inverted on all locals because we found that it is geometrically
	'more convenient to define locals that way
	
	' *Green Button lets all program run
	
Go Retract_Safe
	
Move Workablehight
	
Do		'Button press loop

	'---- Task 1: Pick & Place ----'

    If Sw(1) = On Then	'If White Button is pressed, Task 1 will be executed
		
		Tokens = 3				'number of tokens to place
    	Blocks = 3				'number of blocks to place

		TmReset 1				'Timer starts for Task1
		Print "Task 1 Timer Started"
		
		
    	For TokenID = Tokens To 1 Step -1	'Token Loop
        	Pick_Infeed_Token(TokenID)
        	Alignment_Token()
        	Place_Tray_Token(TokenID)
    	Next TokenID

    	Go Retract_Safe

    	For BlockID = Blocks To 1 Step -1	'Block Loop
       		Pick_Infeed_Block(BlockID)
        	Alignment_Block()
        	Place_Tray_Block(BlockID)
    	Next BlockID

    	Go Retract_Safe
    	
    	Go Home2
    	
    	timeTask1 = Tmr(1)		'Timer ends for Task1
    	Print "Task 1 Completed. Time:" + Str$(timeTask1) + " seconds"
    	
    
     EndIf

	
	
	'---- Task 2: Stacking ----' 	

    If Sw(2) = On Then	'If Blue Button is pressed, Task 2 will be executed

		Tokens = 10				'number of tokens to stack
    	Blocks = 10				'number of blocks to stack

		StackPieces = Tokens + Blocks	'number of pieces to be stacked

		TmReset 2				'Timer starts for Task2
		Print "Task 2 Timer Started"
		
		TokenID = Tokens
		BlockID = Blocks
						
		'Task 2 functions go here
			For i = 1 To StackPieces			'Stacking loop
			
				If (i Mod 2) = 0 Then		'Pick and align block
					Pick_Infeed_Token(TokenID)
					TokenID = TokenID - 1
				Else						'Pick and align token
				   	Pick_Infeed_Block(BlockID)
				   	BlockID = BlockID - 1
				EndIf

				Call Place_Stack(i)				'Place piece being held
			Next i

    	Go Retract_Safe
    	Go Home2

    	timeTask2 = Tmr(2)		'Timer ends for Task2
    	Print "Task 2 Completed. Time:" + Str$(timeTask2) + " seconds"
    
    EndIf

Loop



Fend




'===========================================
'          Shared Functions
'===========================================

Function Pick_Infeed_Token(Tokens_Left As Integer)		'Pick token

    Go Feeder_Token -Z(50 + (Tokens_Left * TokenHeight))

    Integer Infeed_Height
    Infeed_Height = Tokens_Left * TokenHeight
	Print "Infeed token height: ", Infeed_Height

    Go Feeder_Token -Z((Tokens_Left * TokenHeight) + Nozzle_Height_Compensation)
    On 8						'Vacuum nozzle on
    Wait .5
    Go Feeder_Token +X(-2) -Z(50 + (Tokens_Left * TokenHeight))



Fend

Function Pick_Infeed_Block(Blocks_Left As Integer)		'Pick block

    Go Feeder_Block -Z(50 + (Blocks_Left * BlockHeight))

    Integer Infeed_Height
    Infeed_Height = Blocks_Left * BlockHeight
	Print "Infeed token height: ", Infeed_Height

    Go Feeder_Block -Z((Blocks_Left * BlockHeight) + Nozzle_Height_Compensation)
    On 8
    Wait .5
    Go Feeder_Block +X(-2) +Y(-2) -Z(50 + (Blocks_Left * BlockHeight))


Fend

Function Alignment_Token      'Aligns Tokens using inclined surface
	Go Workablehight 		'Back to work position

    Go Drop_Align -Z(40)       'Approching security position
    Go Drop_Align '-Z(10)       'Approach near ramp

    Off 8                         'Turn off vacuum nozzle
    Wait 0.8                      'Accomodation time

    Go Align_Token -Z(Nozzle_Height_Compensation)        'Approach token
    On 8                          'Vacuum nozzle on 
    Wait 0.3

    Go Align_Token -Z(40)

	Go Workablehight 		'Back to work position

Fend

Function Alignment_Block       'Aligns Blocks using inclined surface
	Go Workablehight 		'Back to work position

    Go Drop_Align -Z(40) 		'Approching security position
    Go Drop_Align '-Z(10)		'Approach near ramp

    Off 8						'Turn off vacuum nozzle
    Wait 0.8					'Accomodation time

    Go Align_Block -Z(Nozzle_Height_Compensation)		'Approach block
    On 8						'Vacuum nozzle on
    Wait 0.3

    Go Align_Block -Z(40)

	Go Workablehight 		'Back to work position

Fend


Function Place_Tray_Token(Tokens_Left As Integer)	'Place Token in tray

    Go Pallet(1, 2, Tokens_Left) -Z(20)
    Go Pallet(1, 2, Tokens_Left) -Z(Nozzle_Tray_Height_Compensation)
    Off 8
    Wait .5
    Go Pallet(1, 2, Tokens_Left) -Z(20)


	Go Workablehight 		'Back to work position
	
Fend

Function Place_Tray_Block(Blocks_Left As Integer)	'Place Block in tray

    Go Pallet(1, 1, Blocks_Left) -Z(20)
    Go Pallet(1, 1, Blocks_Left) -Z(Nozzle_Tray_Height_Compensation)
    Off 8
    Wait .5
    Go Pallet(1, 2, Blocks_Left) -Z(20)


	Go Workablehight 		'Back to work position
	
Fend


Function Place_Stack(Index As Integer)		'Stack pieces
	  Go Pile -Z((Index * TokenHeight) + 12)
	  Go Pile -Z((Index * TokenHeight) - 3 + Nozzle_Height_Compensation)
	
	  Off 8       ' VACUUM OFF
	  Wait 0.3
	
	  Go Pile -Z((Index * TokenHeight) + 12)
	
Fend

'=========================================================
' TASK 3 - Tic-Tac-Toe (Robot vs User)
' Epson RC+ 5.0 / RC180 / Epson C3
'
' Buttons:
'   Sw(1) White  = GAME MODE start
'   Sw(2) Blue   = NEXT TURN (user finished move -> robot scans & plays)
'   Sw(3) Orange (Latch) = RESET MODE (priority, cancels game if active)
'
' Pieces:
'   User  = Blocks  -> U( )
'   Robot = Tokens  -> RB( )
'
' Pressure sensor:
'   In(16) = 1 if piece present, 0 if empty
'
' Pallets:
'   Pallet 1, P14, P17, P16, 2, 3       ' storage (row1 blocks, row2 tokens)
'   Pallet 2, TB_P1, TB_P2, TB_P3, 3, 3 ' board 3x3
'
' NOTE:
' - Teach all named points in your robot.
' - Adjust Z_SAFE / Z_TOUCH / Z_LIFT to match your board & sensor.
' - Feeder/align functions are kept as you wrote them.
'=========================================================

'===========================
' Variables
'===========================
Integer Tokens
Integer Blocks
Double TokenHeight
Double BlockHeight
Double Nozzle_Height_Compensation
Double Nozzle_Tray_Height_Compensation

Integer idx
Integer TokenID
Integer BlockID

Integer gameActive
Integer gameOver

Real timeGame
Real timeReset

' TicTacToe arrays (1..9)
Integer U(9)      ' User/Blocks occupancy memory
Integer RB(9)     ' Robot/Tokens occupancy memory
Integer OCC(9)    ' Sensor occupancy

' Motion / sensing heights
Double Z_SAFE
Double Z_TOUCH
Double Z_LIFT

' Pressure input channel
Integer PRESS_IN_CH


'===========================
' Main
'===========================
Function main

    Motor On
    Power High
    Speed 100
    Accel 100, 100
    SpeedS 1800
    AccelS 2500, 2500
    Tool 1

    TokenHeight = 6.0
    BlockHeight = 6.0
    Nozzle_Height_Compensation = 0.0
    Nozzle_Tray_Height_Compensation = 0.0

    PRESS_IN_CH = 16

    ' Tune these for your board + pressure sensor
    Z_SAFE = 25
    Z_TOUCH = 0
    Z_LIFT = 25

    ' Pallets
    Pallet 1, P14, P17, P16, 2, 3
    Pallet 2, TB_P1, TB_P2, TB_P3, 3, 3

    gameActive = 0
    gameOver = 0
    Call Clear_Matrices()

    Go Workablehight CP

    Do

        '=================================================
        ' RESET MODE (Sw3) - priority
        '=================================================
        If Sw(3) = On Then

            gameActive = 0
            gameOver = 0

            TmReset 2
            Print "RESET MODE: scanning + cleanup..."

            Call Scan_Board()
            Call Update_Matrices()

            TokenID = 1
            BlockID = 1

            For idx = 1 To 9
                If RB(idx) = 1 Then
                    Call Pick_From_Board_Cell(idx)
                    Call Return_Token_To_Storage(TokenID)
                    TokenID = TokenID + 1
                ElseIf U(idx) = 1 Then
                    Call Pick_From_Board_Cell(idx)
                    Call Return_Block_To_Storage(BlockID)
                    BlockID = BlockID + 1
                EndIf
            Next idx

            Call Clear_Matrices()

            timeReset = Tmr(2)
            Print "RESET MODE done. Time: " + Str$(timeReset) + " s"

            Wait Sw(3) = Off
        EndIf


        '=================================================
        ' GAME MODE start (Sw1)
        '=================================================
        If Sw(1) = On Then
            gameActive = 1
            gameOver = 0

            TmReset 1
            Print "GAME MODE started. User: make a move, then press BLUE (Sw2)."

            Call Scan_Board()
            Call Update_Matrices()

            Wait Sw(1) = Off
        EndIf


        '=================================================
        ' NEXT TURN (Sw2)
        '=================================================
        If (gameActive = 1) And (Sw(2) = On) Then

            If Sw(3) = On Then
                ' Reset has priority, do nothing here
            ElseIf gameOver = 1 Then
                Print "GAME MODE: game finished. Press RESET (Sw3)."
            Else
                Print "NEXT TURN: scanning board..."
                Call Scan_Board()
                Call Update_Matrices()

                If Check_Win_User() = 1 Then
                    Print "GAME MODE: USER WINS."
                    gameOver = 1
                ElseIf Check_Win_Robot() = 1 Then
                    Print "GAME MODE: ROBOT WINS."
                    gameOver = 1
                Else
                    idx = Choose_Robot_Move()
                    If idx = 0 Then
                        Print "GAME MODE: DRAW."
                        gameOver = 1
                    Else
                        Print "Robot plays cell " + Str$(idx)
                        Call Place_Robot_Token_On_Cell(idx)
                        RB(idx) = 1
                    EndIf
                EndIf

                timeGame = Tmr(1)
                Print "GAME elapsed: " + Str$(timeGame) + " s"
            EndIf

            Wait Sw(2) = Off
        EndIf

    Loop

Fend


'=========================================================
' Board scan
'=========================================================
Function Scan_Board()
    For idx = 1 To 9
        OCC(idx) = Read_Cell_Occupied(idx)
    Next idx
Fend


Function Read_Cell_Occupied(cellIdx As Integer) As Integer
    Integer r, c
    Integer v

    r = ((cellIdx - 1) / 3) + 1
    c = ((cellIdx - 1) Mod 3) + 1

    Go Pallet(2, r, c) +Z(Z_SAFE) CP
    Go Pallet(2, r, c) +Z(Z_TOUCH)
    Wait 0.05

    If In(PRESS_IN_CH) = On Then
        v = 1
    Else
        v = 0
    EndIf

    Go Pallet(2, r, c) +Z(Z_LIFT) CP

    Read_Cell_Occupied = v
Fend


'=========================================================
' Update matrices
'=========================================================
Function Update_Matrices()

    For idx = 1 To 9

        If OCC(idx) = 0 Then
            U(idx) = 0
            RB(idx) = 0
        Else
            If (U(idx) = 0) And (RB(idx) = 0) Then
                ' New piece detected -> assume user (block)
                U(idx) = 1
            EndIf
        EndIf

    Next idx

Fend


Function Clear_Matrices()
    For idx = 1 To 9
        U(idx) = 0
        RB(idx) = 0
        OCC(idx) = 0
    Next idx
Fend


Function Is_Empty(cellIdx As Integer) As Integer
    If (U(cellIdx) = 0) And (RB(cellIdx) = 0) Then
        Is_Empty = 1
    Else
        Is_Empty = 0
    EndIf
Fend


'=========================================================
' TicTacToe move selection (no array parameters)
'=========================================================
Function Choose_Robot_Move() As Integer
    Integer mv

    mv = Find_Winning_Move_Robot()
    If mv <> 0 Then
        Choose_Robot_Move = mv
        Exit Function
    EndIf

    mv = Find_Winning_Move_User()
    If mv <> 0 Then
        Choose_Robot_Move = mv
        Exit Function
    EndIf

    If Is_Empty(5) = 1 Then
        Choose_Robot_Move = 5
        Exit Function
    EndIf

    If Is_Empty(1) = 1 Then
        Choose_Robot_Move = 1
        Exit Function
    EndIf

    If Is_Empty(3) = 1 Then
        Choose_Robot_Move = 3
        Exit Function
    EndIf

    If Is_Empty(7) = 1 Then
        Choose_Robot_Move = 7
        Exit Function
    EndIf

    If Is_Empty(9) = 1 Then
        Choose_Robot_Move = 9
        Exit Function
    EndIf

    For idx = 1 To 9
        If Is_Empty(idx) = 1 Then
            Choose_Robot_Move = idx
            Exit Function
        EndIf
    Next idx

    Choose_Robot_Move = 0
Fend



Function Find_Winning_Move_Robot() As Integer
    Integer winL(8, 3)
    Integer k
    Integer a1, a2, a3

    winL(1, 1) = 1
    winL(1, 2) = 2
    winL(1, 3) = 3

    winL(2, 1) = 4
    winL(2, 2) = 5
    winL(2, 3) = 6

    winL(3, 1) = 7
    winL(3, 2) = 8
    winL(3, 3) = 9

    winL(4, 1) = 1
    winL(4, 2) = 4
    winL(4, 3) = 7

    winL(5, 1) = 2
    winL(5, 2) = 5
    winL(5, 3) = 8

    winL(6, 1) = 3
    winL(6, 2) = 6
    winL(6, 3) = 9

    winL(7, 1) = 1
    winL(7, 2) = 5
    winL(7, 3) = 9

    winL(8, 1) = 3
    winL(8, 2) = 5
    winL(8, 3) = 7

    For k = 1 To 8
        a1 = winL(k, 1)
        a2 = winL(k, 2)
        a3 = winL(k, 3)

        If (RB(a1) + RB(a2) + RB(a3)) = 2 Then
            If Is_Empty(a1) = 1 Then
                Find_Winning_Move_Robot = a1
                Exit Function
            EndIf
            If Is_Empty(a2) = 1 Then
                Find_Winning_Move_Robot = a2
                Exit Function
            EndIf
            If Is_Empty(a3) = 1 Then
                Find_Winning_Move_Robot = a3
                Exit Function
            EndIf
        EndIf
    Next k

    Find_Winning_Move_Robot = 0
Fend

Function Find_Winning_Move_User() As Integer
    Integer winL(8, 3)
    Integer k
    Integer a1, a2, a3

    winL(1, 1) = 1
    winL(1, 2) = 2
    winL(1, 3) = 3

    winL(2, 1) = 4
    winL(2, 2) = 5
    winL(2, 3) = 6

    winL(3, 1) = 7
    winL(3, 2) = 8
    winL(3, 3) = 9

    winL(4, 1) = 1
    winL(4, 2) = 4
    winL(4, 3) = 7

    winL(5, 1) = 2
    winL(5, 2) = 5
    winL(5, 3) = 8

    winL(6, 1) = 3
    winL(6, 2) = 6
    winL(6, 3) = 9

    winL(7, 1) = 1
    winL(7, 2) = 5
    winL(7, 3) = 9

    winL(8, 1) = 3
    winL(8, 2) = 5
    winL(8, 3) = 7

    For k = 1 To 8
        a1 = winL(k, 1)
        a2 = winL(k, 2)
        a3 = winL(k, 3)

        If (U(a1) + U(a2) + U(a3)) = 2 Then
            If Is_Empty(a1) = 1 Then
                Find_Winning_Move_User = a1
                Exit Function
            EndIf
            If Is_Empty(a2) = 1 Then
                Find_Winning_Move_User = a2
                Exit Function
            EndIf
            If Is_Empty(a3) = 1 Then
                Find_Winning_Move_User = a3
                Exit Function
            EndIf
        EndIf
    Next k

    Find_Winning_Move_User = 0
Fend


Function Check_Win_User() As Integer
    Integer winL(8, 3)
    Integer k
    Integer a1, a2, a3

    winL(1, 1) = 1
    winL(1, 2) = 2
    winL(1, 3) = 3

    winL(2, 1) = 4
    winL(2, 2) = 5
    winL(2, 3) = 6

    winL(3, 1) = 7
    winL(3, 2) = 8
    winL(3, 3) = 9

    winL(4, 1) = 1
    winL(4, 2) = 4
    winL(4, 3) = 7

    winL(5, 1) = 2
    winL(5, 2) = 5
    winL(5, 3) = 8

    winL(6, 1) = 3
    winL(6, 2) = 6
    winL(6, 3) = 9

    winL(7, 1) = 1
    winL(7, 2) = 5
    winL(7, 3) = 9

    winL(8, 1) = 3
    winL(8, 2) = 5
    winL(8, 3) = 7

    For k = 1 To 8
        a1 = winL(k, 1)
        a2 = winL(k, 2)
        a3 = winL(k, 3)

        If (U(a1) = 1) And (U(a2) = 1) And (U(a3) = 1) Then
            Check_Win_User = 1
            Exit Function
        EndIf
    Next k

    Check_Win_User = 0
Fend


Function Check_Win_Robot() As Integer
    Integer winL(8, 3)
    Integer k
    Integer a1, a2, a3

    winL(1, 1) = 1
    winL(1, 2) = 2
    winL(1, 3) = 3

    winL(2, 1) = 4
    winL(2, 2) = 5
    winL(2, 3) = 6

    winL(3, 1) = 7
    winL(3, 2) = 8
    winL(3, 3) = 9

    winL(4, 1) = 1
    winL(4, 2) = 4
    winL(4, 3) = 7

    winL(5, 1) = 2
    winL(5, 2) = 5
    winL(5, 3) = 8

    winL(6, 1) = 3
    winL(6, 2) = 6
    winL(6, 3) = 9

    winL(7, 1) = 1
    winL(7, 2) = 5
    winL(7, 3) = 9

    winL(8, 1) = 3
    winL(8, 2) = 5
    winL(8, 3) = 7

    For k = 1 To 8
        a1 = winL(k, 1)
        a2 = winL(k, 2)
        a3 = winL(k, 3)

        If (RB(a1) = 1) And (RB(a2) = 1) And (RB(a3) = 1) Then
            Check_Win_Robot = 1
            Exit Function
        EndIf
    Next k

    Check_Win_Robot = 0
Fend


'=========================================================
' Board pick/place + return
'=========================================================
Function Place_Robot_Token_On_Cell(cellIdx As Integer)
    Pick_Infeed_Token(1)     ' keep your feeder logic; replace "1" with a real counter if you want
    Alignment_Token()
    Call Place_On_Board_Cell(cellIdx)
Fend


Function Place_On_Board_Cell(cellIdx As Integer)
    Integer r, c
    r = ((cellIdx - 1) / 3) + 1
    c = ((cellIdx - 1) Mod 3) + 1

    Go Pallet(2, r, c) +Z(Z_SAFE) CP
    Go Pallet(2, r, c) +Z(Nozzle_Tray_Height_Compensation)
    Wait 0.1
    Off 8
    Wait 0.2
    Go Pallet(2, r, c) +Z(Z_LIFT) CP
Fend


Function Pick_From_Board_Cell(cellIdx As Integer)
    Integer r, c
    r = ((cellIdx - 1) / 3) + 1
    c = ((cellIdx - 1) Mod 3) + 1

    Go Pallet(2, r, c) +Z(Z_SAFE) CP
    Go Pallet(2, r, c) +Z(Z_TOUCH)
    On 8
    Wait 0.2
    Go Pallet(2, r, c) +Z(Z_LIFT) CP
Fend


Function Return_Token_To_Storage(slotIdx As Integer)
    Go Pallet(1, 2, slotIdx) +Z(20)
    Go Pallet(1, 2, slotIdx) +Z(Nozzle_Tray_Height_Compensation)
    Off 8
    Wait 0.2
    Go Pallet(1, 2, slotIdx) +Z(20) CP
Fend


Function Return_Block_To_Storage(slotIdx As Integer)
    Go Pallet(1, 1, slotIdx) +Z(20)
    Go Pallet(1, 1, slotIdx) +Z(Nozzle_Tray_Height_Compensation)
    Off 8
    Wait 0.2
    Go Pallet(1, 1, slotIdx) +Z(20) CP
Fend


'=========================================================
' ==== YOUR ORIGINAL FEEDER / ALIGNMENT FUNCTIONS (kept) ====
'=========================================================
Function Pick_Infeed_Token(Tokens_Left As Integer)

    Go Feeder_Token +Z(50 + (Tokens_Left * TokenHeight))

    Integer Infeed_Height
    Infeed_Height = Tokens_Left * TokenHeight
    Print "Infeed token height: ", Infeed_Height

    Go Feeder_Token +Z((Tokens_Left * TokenHeight) - 6.2 + Nozzle_Height_Compensation)
    On 8
    Wait .25
    Go Feeder_Token +Y(-7) +Z(50 + (Tokens_Left * TokenHeight)) CP

Fend


Function Pick_Infeed_Block(Blocks_Left As Integer)

    Go Feeder_Block +Z(50 + (Blocks_Left * BlockHeight))

    Integer Infeed_Height
    Infeed_Height = Blocks_Left * BlockHeight
    Print "Infeed block height: ", Infeed_Height

    Go Feeder_Block +Z((Blocks_Left * BlockHeight) - 6 + Nozzle_Height_Compensation)
    On 8
    Wait .25
    Go Feeder_Block +X(-2) +Y(-5) +Z(50 + (Blocks_Left * BlockHeight)) CP

Fend


Function Alignment_Token
    Go Workablehight CP

    Go Drop_Align -Z(40) CP
    Go Drop_Align

    Off 8
    Wait 0.3

    Go Align_Token -Z(Nozzle_Height_Compensation)
    On 8
    Wait 0.3

    Go Align_Token -Z(40) -X(10) -Y(10) CP

    Go Workablehight CP
Fend


Function Alignment_Block
    Go Workablehight CP

    Go Drop_Align -Z(40) CP
    Go Drop_Align

    Off 8
    Wait 0.3

    Go Align_Block -Z(Nozzle_Height_Compensation)
    On 8
    Wait 0.3

    Go Align_Block -Z(40) -X(10) -Y(10) CP

    Go Workablehight CP
Fend
