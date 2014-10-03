Battleship-Game
===============
/*
**********************************************************************
**
**  Battleship.cpp
**
**  Author:  Bary W Pollack
**
**  A simple interactive "Battleship" game
**
**  This program plays the game of "Battleship" on a 5 x 5 grid.
**  The single battleship is randomly placed by the program, either
**  horizontally or vertically in the grid. The player's job is to
**  fire missiles at grid cells until the battleship has been sunk.
**
**  The game could be extended quite easily to support a larger grid
**  along with multiple warships.
**
**  Also, one could consider creating various computer "strategies"
**  for firing missles in a version of the game where a human places
**  the ships and the computer attempts to sink them.
**
**  Modified:  04/09/99  -  created
**  Modified:  04/10/99  -  DisplayState changed to meet specs
**  Modified:  04/11/99  -  added captured output
**  Modified:  10/18/10  -  updated to VS 2008
**
**********************************************************************
*/

#include <iostream>
#include <time.h>       // for seeding the random number generator

using namespace std;

//  Random() and Randomize() have been "borrowed" from stdlib.h
//  since some C++ IDE's don't contain one or the other routine.
//
//      Random(num) returns a random int in the range:  0 -(num-1)
//      Randomize seeds the pseudo-random number generator

#define Random(num)(int)(((long) rand() *(num))/(RAND_MAX+1))

void Randomize(void) { srand((unsigned) time(NULL)); }


/*
**********************************************************************
**  Global constants and types
**********************************************************************
*/

// Bool, False, True used since some C++ compilers don't support 'bool'
enum Bool  {  False = 0, True = 1  };

const int BRD_SIZE = 5;             // size of the board
const int SHIP_LTH = BRD_SIZE-1;    // battleship length

enum CELL                           // contents of the board
{                                   //(for display purposes)
    EMPTY = '_',                    // empty cell - ocean
    HIT   = 'X',                    // battleship hit by a missle
    MISS  = 'O'                     // contains an exploded missle
};


/*
**********************************************************************
**  Class declarations for Battleship and BattleshipGame
**********************************************************************
**  A Battleship is a ship that is positioned on the board in a
**  fixed random position and orientation(horizontal or vertical).
**
**  IsHit is the battleship's way to tell you if a missle fired to
**  location(y,x) hits the ship.
**
**  IsSunk is the battleship's way to tell you if it is sunk / afloat.
**
**  Display is a routine used during development that allows you to
**  see the(private) battleship data.
**  
**********************************************************************
*/

class Battleship
{
public:
    Battleship(void);
    Bool IsHit(const int y, const int x);
    Bool IsSunk(void);
    void Display(void);

private:
    struct ShipData
    {
	int  y, x;
	Bool bHit;
    } shipData[SHIP_LTH];
};

/*
**********************************************************************
**  A BattleshipGame is a game having a square board that contains
**  a single(hidden) battleship. The game keeps track of the number
**  of times you play, the number of missiles you fire, and the number
**  of missiles that hit the battleship.
**
**  Play currently plays a single game. Play is responsible for 
**  acquiring firing locations from the user, updating the board,
**  and loops until the game is over.
**
**  DisplayState is used by Play to display the current board state.
**********************************************************************
*/

class BattleshipGame
{
public:
    BattleshipGame(const int nGameNumber);
    void Play(void);
    void DisplayState(const char *sMsg);

private:
    CELL        board[BRD_SIZE+1][BRD_SIZE+1];
    Battleship  ship;
    int         nGameNumber;
    int         nMissilesFired;
    int         nHits;
};

/*
**********************************************************************
**  Implementation of class Battleship
**********************************************************************
**  Constructor - randomly orients the battleship; randomly
**                positions the battleship(but NOT on the board --
**                the battleship's position is HIDDEN from the game)
**********************************************************************
*/

Battleship::Battleship(void)
{
    // offset into the board:  flush against left or right side
    int nOffset = Random(2) + 1;   // will be 1 or 2

    // battleship will be either horizontal or vertical; not diagonal
    if (Random(2))
    {       // vertical battleship
	int nI = Random(BRD_SIZE) + 1;
	for (int j = 0; j < SHIP_LTH; j++)
	{
	    shipData[j].y = j + nOffset;
	    shipData[j].x = nI;
	    shipData[j].bHit = False;
	}
    }
    else
    {       // horizontal battleship
	int nJ = Random(BRD_SIZE) + 1;
	for (int i = 0; i < SHIP_LTH; i++)
	{
	    shipData[i].y = nJ;
	    shipData[i].x = i + nOffset;
	    shipData[i].bHit = False;
	}
    }
}

/*
**********************************************************************
**  IsHit - returns True if a missile fired at(x,y) hits the 
**          battleship; otherwise it returns False.
**          In case of a "hit," the hit is registered in the shipData.
**********************************************************************
*/

Bool Battleship::IsHit(const int y, const int x)
{
    Bool bIsHit = False;
    for (int i = 0; i < SHIP_LTH; i++)
	if (shipData[i].y == y && shipData[i].x == x)
	{
	    shipData[i].bHit = bIsHit = True;
	    break;
	}
	return bIsHit;
}

/*
**********************************************************************
**  IsSunk -- returns True if the battleship is sunk(hit in all
**            positions; it returns False otherwise
**********************************************************************
*/

Bool Battleship::IsSunk(void)
{
    Bool bIsSunk = True;
    for (int i = 0; i < SHIP_LTH; i++)
	if (! shipData[i].bHit)
	    {  bIsSunk = False;  break;  }
	return bIsSunk;
}

/*
**********************************************************************
**  This debug routine was used during development so that we could
**  see where the battleship actually is located and can track the
**  shipData as the game is played.
**********************************************************************
*/

void Battleship::Display(void)
{
    cout << "Battleship: ";
    for (int i = 0; i < SHIP_LTH; i++)
	cout << "(" << shipData[i].y
	     << " "  << shipData[i].x
	     << (shipData[i].bHit ? " *)" : " .)");
    cout << endl;
}

/*
**********************************************************************
**  Implementation of class BattleshipGame
**********************************************************************
**  The constructor sets up the game. This includes initialization
**  of the entire board as well as other game-related data such as 
**  the game number, number of missiles fired and hits.
**********************************************************************
*/

BattleshipGame::BattleshipGame(const int nGameNumber)
{
    this->nGameNumber = nGameNumber;

    for (int j = 1; j <= BRD_SIZE; j++)
	for (int i = 1; i <= BRD_SIZE; i++)
	    board[j][i] = EMPTY;

    nMissilesFired = nHits = 0;
}

/*
**********************************************************************
**  Play implements one "game:"  repeatedly asking for pairs of 
**  missile coordinates, updating the board, tracking the number of 
**  fired missiles and the number of hits, redisplaying the board, 
**  until the battleship is sunk.
**
**  Some simple range-checking is done on the values of y and x.
**
**  Note:  a y or an x coordinate <= zero aborts the current game.
**
**  Coordinate convention:(y x), with y increasing downwards and
**                                    x increasing to the right
**
**                             x -->
**
**                         1 2 3 4 5
**                      1  . . . . .
**                      2  . . . . .
**                  y   3  . O O X .
**                  |   4  . . . X .
**                  V   5  . . . . .
**
**  Note:  the 0-row and 0-column of the board(array) are "ignored"
**         so that we can use "natural" values for indices.
**
**********************************************************************
*/

void BattleshipGame::Play(void)
{
    while (! ship.IsSunk())
    {
	int  y, x;

	DisplayState("The current situation:");

	// There is a reasonable argument to make the acquisition of the
	// user's input process into a separate routine. And, in general,
	// you probably should. But, in this program there's so little to
	// do that there isn't much of an advantage to do so. So I didn't.

	cout << "Fire missile to coordinates (y x): ";
	x = y = -1;
	cin >> y;
	if (y <= 0)     // abort?
	    break;
	cin >> x;
	if (x <= 0)     // abort?
	    break;

	if (x > BRD_SIZE || y > BRD_SIZE)
	{
	    cout << "Each coordinate must be in the range 1-" << BRD_SIZE << endl
		 << "Please try again..." << endl << endl;
	    continue;
	}
	if (board[y][x] != EMPTY)
	    cout << "That's foolish!  You've already fired at("
		 << y << "," << x << ")" << endl;

	// now, enter the user's data and record the results

	++nMissilesFired;
	if (ship.IsHit(y, x))
	{
	    board[y][x] = HIT;
	    ++nHits;
	}
	else
	    board[y][x] = MISS;

	// protection against infinite loops in IDEs like Borland...
	// if (nMissilesFired > BRD_SIZE*BRD_SIZE)  break;  //FOO
    }

    if (ship.IsSunk())
	DisplayState("Down she goes!");
}

/*
**********************************************************************
**  Display the state of the game: board, missiles fired, hits...
**********************************************************************
*/

void BattleshipGame::DisplayState(const char *sMsg)
{
    int  j, i;

    cout << endl << endl << sMsg << endl << endl << ' ';

    for (i = 1; i <= BRD_SIZE; i++)
	cout << ' ' << i;
    cout << endl;

    for (j = 1; j <= BRD_SIZE; j++)
    {
	cout << j << ' ' << char(board[j][1]);
	for (i = 2; i <= BRD_SIZE; i++)
	    cout << '|' << char(board[j][i]);
	cout << endl;
    }

    // ship.Display();             //FOO - for development only...
    cout << endl
	 << "Game: " << nGameNumber << "   "
	 << "Missle Count: " << nMissilesFired << "   "
	 << "Hits: " << nHits
	 << endl << endl;
}

/*
**********************************************************************
**  The main routine plays multiple games of Battleship.
**********************************************************************
*/

int main(void)
{
    Randomize();

    cout << endl << "* * B A T T L E S H I P * *" << endl;

    char ch;
    int  nGameNumber = 0;
    do              // play one game
    {
	int nLoopCount = 3;	// deal with bad input
	BattleshipGame game(++nGameNumber);
	game.Play();
	do          // ask user about playing again;
	{           // again, this could be a separate routine...
	    cout << "Game Over!" << endl << endl
		 << "Would you like to play again (y/n)? ";
	    cin >> ch;
	    ch = (char) tolower(ch);
	    if (--nLoopCount < 0)
		break;
	} while (ch != 'y' && ch != 'n');

    } while (ch == 'y' && nGameNumber < 20);	// deal with bad input

    cout << endl << "Thanks for playing..." << endl << endl;

    return 0;
}
