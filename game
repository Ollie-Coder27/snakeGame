/*
gcc final.c -lrt -lcurses -o "final.exe"  
*/

//Headers
#include <curses.h>
#include <ncurses.h>
#include <sys/select.h>
#include <unistd.h>
#include <stdlib.h>
#include <aio.h>
#include <sys/ioctl.h>
#include <stdio.h>  //for getting terminal dimensions
#include <string.h>
#include <signal.h>
#include <time.h>
#include <ctype.h> //lower case user in for main menu

//Define
#define DELAY 300000
#define rows "-"
#define cols "|"

//Structures
struct snake {
  int xpos, ypos;
  int direction;
  int length;
  int speed;
  char symbol;
  int xtail[200], ytail[200];
};
struct trophy {
  int value;
  int lifetime;
  int xpos, ypos;
  char symbol;
};

//Classes
void startGame();
void defineWindow();
void createPit();
void createSnake();
void moveKeys(int);
void moveSnake();
void main_menu();
void winConditions();
void endGame();
void createTrophy();
void snakeEats();
void scorePrint();
void doubleBack();
//void updateSnake();

//Variables
int perimeter = 0;
int yAxis, xAxis = 0;
int topRow, bottomRow, leftCol, rightCol;
static int tolength = 2; // total snake length -1 (for head)
static int lose = 0; // which message
static int done = 0; // end game
static int tempDir = 'd';
static int activeTrophy = 0;
static int score = 0;

struct snake makeSnake;
struct trophy makeTrophy;

//Start the Snake
//Snake head and tail
void snakehead() {
  //create the head at the start and make it this symbol
  mvaddch(makeSnake.ypos, makeSnake.xpos, makeSnake.symbol);
  curs_set(0); //hides cursor
}

void setTailPos(int place, int newy, int newx) {
  makeSnake.ytail[place] = newy;
  makeSnake.xtail[place] = newx;
}

//cover the last character with a space to 'move' the snake
void newTail() {
  mvaddch(makeSnake.ytail[makeSnake.length], makeSnake.xtail[makeSnake.length], ' ');
  setTailPos(makeSnake.length, -1, -1);
}

//move snake through
void tailShift() {
  for (int i = makeSnake.length; i > 0; i--) {
    setTailPos(i, makeSnake.ytail[i - 1], makeSnake.xtail[i - 1]);
  }
  setTailPos(0, makeSnake.ypos, makeSnake.xpos);
  newTail();
}

//Random integer
//Get a random value for trophy (random integer function)
int randomInt(int min, int max) {
  //use time as seed to guarantee new values
  time_t t;
  srand((unsigned) time( & t));
  return (rand() % (max - min + 1)) + min;
}

//Main / Main_Menu, Game Start + Set Up

//main menu to allow player to play the game, view previous scores, or exit
int main() {
  char userIn[20];

  // printf("\033[0;32m"); // Set text color to green
  start_color();
  init_pair(1, COLOR_GREEN, COLOR_BLACK); //Green Snake, black background
  attron(COLOR_PAIR(1));

  int midX, midY;

  midX = (COLS / 4) + 25;
  midY = LINES / 4;

  printf("                         888              \n");
  printf("                         888              \n");
  printf("                         888              \n");
  printf(" .d8888b 88888b.  8888b. 888  888 .d88b.  \n");
  printf(" 88K     888 88b    88b888 .88Pd8P  Y8b \n");
  printf(" Y8888b.888  888.d888888888888K 88888888 \n");
  printf("      X88888  888888  888888 88bY8b.     \n");
  printf("  88888P 888  888Y888888888  888 Y8888\n");


  // THIS WOULD BE FUNNY
  /*
  mvprintw(midY + 10, midX, "                        Give us an A               \n");

  mvprintw(midY + 11, midX, "                         Please              \n");

  mvprintw(midY + 12, midX, "                         Teacher              \n");*/
  mvprintw(midY + 10, midX, "score: %d\n snake length: %d", score, makeSnake.length);
  printf("\nWelcome to snake! Please enter your fascination\n\n");
  printf("                         Play              \n");
  //printf("                         See High Score              \n");
  printf("                         Instuctions (& Exit)     \n");
  printf("                         Quit              \n");
  //Turn off color pair
  attroff(COLOR_PAIR(1));

  //changed to get a string/char - O
  scanf("%[^\n]s", userIn);

  //make all letters lowercase for ease of compare - O
  for (int index = 0; index < strlen(userIn); index++) {
    userIn[index] = tolower(userIn[index]);
  }

  //changed to string compare - O
  if ((strcmp(userIn, "play") == 0) || (strcmp(userIn, "p") == 0) || (strcmp(userIn, "go") == 0)) {
    startGame();
    perimeter = yAxis + xAxis;
    // 1-4 changed to 1-3 is triggering double back
    int randSnakeDirection = randomInt(1,3);
    if(randSnakeDirection == 1){makeSnake.direction = 'N';}
    else if (randSnakeDirection == 2){makeSnake.direction = 'E';}
    else if (randSnakeDirection == 3){makeSnake.direction = 'S';}
    //else if (randSnakeDirection == 4){makeSnake.direction = 'W';} // will trigger double back, got rid of
        
    //Author: Jahmaro Gordon
    while (done != -1) {
      //Creates a timer for user input so program doesn't wait till key is pressed
      timeout(DELAY / 1000);
      int ch = wgetch(stdscr);
      if (ch != ERR) {
        moveKeys(ch); //gets snake direction
      }
            moveSnake();            
    }
    endGame();
  } /*else if ((strcmp(userIn, "highscore") == 0) || (strcmp(userIn, "high") == 0) || (strcmp(userIn, "score") == 0) || (strcmp(userIn, "see highscore") == 0)) {
    // view high score
    printf("Work in progress...");
    //High score scrapped
  }*/ else if((strcmp(userIn, "instuctions") == 0) || (strcmp(userIn, "i") == 0)){
    //instructions
    printf("To move the snake use either the arrow keys or wasd\n");
    printf("To end the game use control + c\n");
    printf("Goals: 1) Collect as many trophies as possible (You win when you are half the length of the perimeter)\n");
    printf("       2) Do not hit the boarder nor yourself\n");
    sleep(3);
    printf("Thank you, have a nice day!\n");
    exit(1);
  }else if ((strcmp(userIn, "quit") == 0) || (strcmp(userIn, "q") == 0) || (strcmp(userIn, "done") == 0)) {
    //quit game
    printf("Thank you, have a nice day!\n");
    exit(1);
  } else {
    printf("That was not an option. Thank you, have a nice day!\n");
    exit(1);
  }
  return 0;
}

//start everything for the game
void startGame() {
  defineWindow();

  initscr(); // determines the screen (for set-up)
  noecho(); // stops characters from being doubled
  crmode(); // No buffering mode 
  keypad(stdscr, TRUE); //enables arrow keys to be used

  createSnake();
  createTrophy();
  refresh();
  createPit();
}

//defines the terminal size for positions and pit
void defineWindow() {
  //printf("\n Define Window \n");
  //define walls
  struct winsize wbuf;
  if (ioctl(0, TIOCGWINSZ, & wbuf) != -1) {
    topRow = 1;
    bottomRow = wbuf.ws_row - 2;
    leftCol = 1;
    rightCol = wbuf.ws_col - 2;
  }

  //define the size of the window as the screen size
}

//builds the pit from the variables defined above
void createPit() {
  //build walls
  int width = rightCol - leftCol + 2;
  int height = bottomRow - topRow + 2;
  //build top and bottom walls
  for (int p = 0; p < width; p++) {
    //top
    move(topRow - 1, leftCol + p);
    addstr(rows);
    //bottom
    move(bottomRow + 1, leftCol + p);
    addstr(rows);
  }
  //build sides
  for (int p = 0; p < height; p++) {
    //left
    move(topRow - 1 + p, leftCol - 1);
    addstr(cols);
    //right
    move(topRow + p, rightCol + 1);
    addstr(cols);
  }
  //update screen to create it
  refresh();

}

//creates the snake
void createSnake() {
  xAxis = COLS / 2;
  yAxis = LINES / 2;

  //initialize snake
  makeSnake.symbol = 'D';
  makeSnake.xpos = xAxis;
  makeSnake.ypos = yAxis;
  snakehead();

  makeSnake.length = tolength;

  //get snake from length ^^
  for (int i = 1; i <= makeSnake.length; i++) {
    makeSnake.ytail[i - 1] = makeSnake.ypos; //same lines
    makeSnake.xtail[i - 1] = makeSnake.xpos - i; //coves xAxis-1, xAxis-2, xAxis-3, xAxis-4
    mvaddch(makeSnake.ytail[i - 1], makeSnake.xtail[i - 1], makeSnake.symbol);
  }
}

//Create Trophy
void createTrophy(){
    //temp values for testing
    makeTrophy.value = randomInt(1, 9);
    makeTrophy.lifetime = randomInt(1, 9) * 10;
    makeTrophy.symbol = makeTrophy.value + '0';
    //active trophy is 0
    while(activeTrophy == 0){
        makeTrophy.xpos = randomInt(leftCol, rightCol);
        makeTrophy.ypos = randomInt(topRow, bottomRow);
        mvaddch(makeTrophy.ypos, makeTrophy.xpos, makeTrophy.symbol);
        activeTrophy++;
        
    }
}

//What happens when the snake eats a trophy
void snakeEats(){
  if ((makeSnake.xpos == makeTrophy.xpos) && (makeSnake.ypos == makeTrophy.ypos)) {
    score = score + makeTrophy.value;
    activeTrophy = 0;
    makeSnake.length = makeSnake.length + makeTrophy.value;
    makeSnake.speed = makeSnake.length;
  }else{
    makeTrophy.lifetime--;
    if(makeTrophy.lifetime <= 0){
        mvaddch(makeTrophy.ypos, makeTrophy.xpos, ' ');
        activeTrophy = 0;
        createTrophy();
    }

  }   
}

//Move Snake
//moving the snake input
void moveKeys(int signum) {
  //printf("Move Keys in");
  int c;
  switch (signum) { //ascii for user input
  case KEY_UP:
  case 'w':
    makeSnake.direction = 'N';
    break;
  case KEY_DOWN:
  case 's':
    makeSnake.direction = 'S';
    break;
  case KEY_LEFT:
  case 'a':
    makeSnake.direction = 'W';
    break;
  case KEY_RIGHT:
  case 'd':
    makeSnake.direction = 'E';
    break;
  default:
    break;
  }
    moveSnake(); 
}

//snake movement
void moveSnake() {
  int delay = makeSnake.length;
  switch (makeSnake.direction) {
  case 'N':
    tailShift();
    makeSnake.ypos--;
    break;
  case 'S':
    tailShift();
    makeSnake.ypos++;  
    break;
  case 'E':
    tailShift();
    makeSnake.xpos++;
    break;
  case 'W':
    tailShift();
    makeSnake.xpos--;
    break;
  }
  snakehead();
  winConditions();
  doubleBack();
  snakeEats();
  
  sleep(delay / 100000); // snake time delay implementation 
  refresh();
}

//check snake position and if it does or not eat a trophy - Oliver
void winConditions() {
  //check border
  if ((makeSnake.xpos < leftCol) || (makeSnake.xpos > rightCol)) {
    //lose game
    lose = 1;
    done = -1; 
  } else if ((makeSnake.ypos < topRow) || (makeSnake.ypos > bottomRow)) {
    //lose game
    lose = 1;
    done = -1; 
  } else if (makeSnake.length == (perimeter / 2)) { // check win condition
    //check win
    lose = 2;
    done = -1; 
  } else {
    //keep playing
    done = 0;
  }
}

//make sure the snake can not hit itself
void doubleBack(){
      //check is snake doubles back
 for(int temp = 0; temp < makeSnake.length; temp++){      
    if((makeSnake.ypos == makeSnake.ytail[temp]) && (makeSnake.xpos == makeSnake.xtail[temp])){
        done = -1;
        lose = 1;
    }
 }
}

//ends the game, presents the score, and clears the window
void endGame() {
  endwin();
  scorePrint();
}

//display the score at the end (dependent on the scenario)
void scorePrint() {
    if(lose == 2){
        printf("\n\nYou win! congratulations!! Here is your score: %d \n\n", score);
    }else if (lose == 1){
        printf("\n\nSorry but you lose! Here is your score: %d \n\n", score);
    }else{
        printf("\n\nYou have exited the game!\n\n");
    }
}
