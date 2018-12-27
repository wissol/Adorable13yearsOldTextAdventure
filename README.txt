#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <ctype.h>

/* =====================================================================================================

********************************************
            
            Halloween Ball

                  or

  the Adorable 13 years old text adventure 

*********************************************

- Started: December 2018
- Status: Early development
- Being a programming exercise by Miguel de Luis Espinosa
- Intending to create a basic, simple, short IF Text Adventure in the 8-bit tradition
  (think Zork, Infocom...)
- What's in a name:
   a) "13 years old text adventure" refers to putting myself in the mood of just doing something simple that works and have no 
      concerns for great coding skills and a lot of important computing things that would get in the way of fun.
   b) "Adorable" was just part of a randomly generated repository name 

 Constraints
=============

  - The parser would understand up to three items: Verb + Predicate_1 + Predicate_2
  - Each of these items may be represented by one or, at most, 2 words 
  - No prepositions, articles, and whathaveyou 
  so "examine the red bear" should be understanded by the parser as Verb "examine" Predicate_1 "red bear"
  "examine the big red bear" would not work 

===================================================================================================== */


/* Types */

typedef unsigned char          uint8_t;

/* Constants */

#define SHOW_ACTUAL_ROOM          show_room(&MY_ROOM,0)
#define NO_VERB                   255
#define NOT_A_VERB                254
#define ERROR_STRING              "\n- %s h for help\n"
#define TITLE_STRING              "\n..:: %s ::..\n\n" 
#define VERB_HELP_STRING          "\t%15s %1s %s\n"
#define TOTAL_NUMBER_OF_ROOMS     13
#define NOWHERE                   TOTAL_NUMBER_OF_ROOMS * 2
#define MY_ROOM                   rooms[actual_room_index]
#define TOTAL_NUMBER_OF_THINGS    24
#define INVENTORY_LOCATION        NOWHERE + 1 
#define NORTH                     0
#define EAST                      1
#define SOUTH                     2
#define WEST                      3
#define UP                        4
#define DOWN                      5
#define MAX_WORD_SIZE            15
#define IS_DECORATION             2   // Things not to be carried, like cobwebs, rats and the like
#define CHARACTER_DESCRIPTION    "As you examine yourself a sinister voice comes to your mind. \"My dearest, you look lovely so afraid\""

/* Global variables */

size_t terminal_size;
uint8_t game_on;

char * player_input = NULL;
char verb[MAX_WORD_SIZE];
char first_word[MAX_WORD_SIZE];
char second_word[MAX_WORD_SIZE];

/* Rooms  */

uint8_t actual_room_index = 0;
uint8_t went_to_a_new_room = 1;

struct room {
  char * l_desc;
  char * s_desc;
  uint8_t north;
  uint8_t east;
  uint8_t west;
  uint8_t south;
  uint8_t up;
  uint8_t down;
  uint8_t visited;
};

typedef struct room ROOM;

// room methods 
ROOM new_room(char * short_description, 
              char * long_description, 
              uint8_t north_exit_index,
              uint8_t east_exit_index,
              uint8_t south_exit_index,
              uint8_t west_exit_index,
              uint8_t up_exit_index,
              uint8_t down_exit_index
              );

void show_room(ROOM * this_room, uint8_t compulsory_ldesc);
u_int8_t go(uint8_t direction);

ROOM new_room(
  char * short_description, 
  char * long_description, 
  uint8_t north_exit_index,
  uint8_t east_exit_index,
  uint8_t south_exit_index,
  uint8_t west_exit_index,
  uint8_t up_exit_index,
  uint8_t down_exit_index);

void game_start();

/* Things  */
struct thing {
  char * l_desc;
  char * s_desc;
  u_int8_t location;
  u_int8_t fixed;
  };

typedef struct thing THING;

THING things[TOTAL_NUMBER_OF_THINGS];

void show_things_in_this_room(uint8_t room_index);

u_int8_t get_something(char * stringo);
u_int8_t drop_something(char * stringo);
u_int8_t examine_something(char * stringo);

u_int8_t examine_something(char * stringo){
  u_int8_t found_examinandum = 0;
  for(u_int8_t i=0; i<TOTAL_NUMBER_OF_THINGS; i++){    
    if(things[i].location == INVENTORY_LOCATION || things[i].location == actual_room_index ){
      if( strcmp(stringo, "all")==0 || strcmp(stringo, things[i].s_desc)==0){
        found_examinandum = 1;
        printf(TITLE_STRING,things[i].s_desc);
        printf("%s\n", things[i].l_desc);
      }
    }
  }  
  return found_examinandum;
}

u_int8_t drop_something(char *stringo){
  u_int8_t dropped_it = 0;
  for(u_int8_t i=0; i<TOTAL_NUMBER_OF_THINGS; i++){
    if(things[i].location == INVENTORY_LOCATION ){
      if(strcmp(stringo,"all" ) == 0 || strcmp(stringo,things[i].s_desc) == 0){
        things[i].location = actual_room_index;
        dropped_it = 0;
      } 
    }
  }
  return dropped_it;
};

u_int8_t get_something(char * stringo)
{
  u_int8_t found_something = 0;
  for(u_int8_t i=0; i<TOTAL_NUMBER_OF_THINGS; i++){
    if(things[i].location == actual_room_index){
      if(strcmp(stringo, "all")==0 || strcmp(stringo, things[i].s_desc)==0 )
      {
        if(things[i].fixed){
          if(things[i].fixed==IS_DECORATION){
            printf("\n %s is not something you pick up.\n", things[i].s_desc);
          } else {
            printf("\n %s cannot be taken.\n", things[i].s_desc);
          }
        } else {
          things[i].location = INVENTORY_LOCATION;
          printf("\n %s picked\n",things[i].s_desc);
          found_something = 1;
        }
      }
    }
  }
  return found_something;
};

void show_things_in_this_room(uint8_t room_index){
  char output_string[200] = "";
  char aux_string[200] = "";
  for(u_int8_t i=0; i<TOTAL_NUMBER_OF_THINGS; i++){
    if(things[i].location == room_index && things[i].fixed != IS_DECORATION){
      if(output_string[0] != 0){ //we have found something
        sprintf(aux_string, ", %s", things[i].s_desc);
        strcat(output_string, aux_string);
        } else {
          sprintf(aux_string, " %s", things[i].s_desc);
          strcat(output_string, aux_string);
        }
    }
  }
  if(output_string[0]){
    printf("- Things:");
    printf("%s.", output_string);
  } else { printf("Nothing important here.");}
}

ROOM rooms[TOTAL_NUMBER_OF_ROOMS];

u_int8_t go(uint8_t direction){
  switch(direction){
    case NORTH: 
      if(MY_ROOM.north != NOWHERE){
        MY_ROOM.visited = 1;
        went_to_a_new_room = 1;
        actual_room_index = MY_ROOM.north; 
        return 1;
      }
      else
      {
        printf("You can't go north\n");
        return 0;
      }
    case EAST: 
      if(MY_ROOM.east != NOWHERE){
        MY_ROOM.visited = 1;
        went_to_a_new_room = 1;
        actual_room_index = MY_ROOM.east; 
        return 1;
      }
      else
      {
        printf("You can't go east\n");
        return 0;
      }
    case SOUTH: 
      if(MY_ROOM.south != NOWHERE){
        MY_ROOM.visited = 1;
        went_to_a_new_room = 1;
        actual_room_index = MY_ROOM.south; 
        return 1;
      }
      else
      {
        printf("You can't go south\n");
        return 0;
      }   
    case WEST:
      if(MY_ROOM.west != NOWHERE){
        MY_ROOM.visited = 1;
        went_to_a_new_room = 1;
        actual_room_index = MY_ROOM.west; 
        return 1;
      }
      else
      {
        printf("You can't go west, young man\n");
        return 0;
      }
    case UP: 
    if(MY_ROOM.up != NOWHERE){
        MY_ROOM.visited = 1;
        went_to_a_new_room = 1;
        actual_room_index = MY_ROOM.up; 
        return 1;
      }
      else
      {
        printf("You can't go up\n");
        return 0;
      }
    case DOWN:
    if(MY_ROOM.down != NOWHERE){
        MY_ROOM.visited = 1;
        went_to_a_new_room = 1;
        actual_room_index = MY_ROOM.down; 
        return 1;
      }
      else
      {
        printf("You can't go down\n");
        return 0;
      }
  }

}

ROOM new_room(char * short_description, 
              char * long_description, 
              uint8_t north_exit_index,
              uint8_t east_exit_index,
              uint8_t south_exit_index,
              uint8_t west_exit_index,
              uint8_t up_exit_index,
              uint8_t down_exit_index)
              {
  ROOM n_room;
  n_room.s_desc = short_description;
  n_room.l_desc = long_description;
  n_room.north  = north_exit_index;
  n_room.south  = south_exit_index;
  n_room.west   = west_exit_index;
  n_room.east   = east_exit_index;
  n_room.up     = up_exit_index;
  n_room.down   = down_exit_index; 
  n_room.visited = 0;
  return n_room;
}

void show_room(ROOM * this_room, uint8_t compulsory_ldesc){
  printf(TITLE_STRING, this_room->s_desc);
  if(compulsory_ldesc==1 || (this_room->visited==0)){
    printf("\t%s\n",this_room->l_desc);
  }
  printf("\n- Exits: %s%s%s%s%s%s\n", 
    this_room->north != NOWHERE ? "n " : "", 
    this_room->east != NOWHERE ? "e " : "",
    this_room->south != NOWHERE ? "s " : "",
    this_room->west != NOWHERE ? "w " : "",
    this_room->up != NOWHERE ? "u " : "",
    this_room->down != NOWHERE ? "d" : ""    
    );
  printf("\n");
  show_things_in_this_room(actual_room_index);
  printf("\n\t\t");
  for(uint8_t i=1; i<60; i++)
    printf("-");
  printf("\n");
}

/* 
Input - Ouput - Parser 
*/

void get_player_input();
u_int8_t verb_hash();
void parse_player_input();
char * underlines(char * string_to_underline);

char * underlines(char * string_to_underline){
  uint8_t i=0;
  char * underline_string;
  underline_string[0] = 0;
  while(string_to_underline[i]!=0)
    {
      underline_string[i++] = '-';
    }
  underline_string[++i] = 0;
  return underline_string;
  };

void get_player_input(){
  printf("\n> ");
  getline(&player_input, &terminal_size, stdin);
  for(int i=0; i < MAX_WORD_SIZE; i++){
    verb[i] = 0;
    first_word[i] = 0;
    second_word[i] = 0;
    //third_word[i] = 0;
  }  
  u_int8_t player_input_index = 0;
  u_int8_t word_index = 0;
  u_int8_t number_of_words = 0;
  u_int8_t previous_char_space = 0;
  while(player_input[player_input_index] != 0 && player_input[player_input_index] !='\n')
  { 
    char chr = player_input[player_input_index];
    if(isalpha(chr))
      {
        chr = tolower(chr);
        switch(number_of_words){
        case 0:
          verb[word_index] = chr;
          break;
        case 1:
          first_word[word_index] = chr;
          break;
        case 2:
          second_word[word_index] = chr;
        }
        previous_char_space = 0;
        word_index++;
      } else {
        if(isspace(chr)){
          if(previous_char_space){
                        
          } else {
            number_of_words++;
            word_index=0;
            previous_char_space=1;
          }
        }
        
      }   
    player_input_index++;
  } 
  printf("%s - %s - %s", verb, first_word, second_word);
}

void show_inventory();

void show_inventory(){
  char output_string[200] = "";
  char aux_string[200] = "";
  for(u_int8_t i=0; i<TOTAL_NUMBER_OF_THINGS; i++){
    if(things[i].location == INVENTORY_LOCATION){
      sprintf(aux_string, "\t* %s\n", things[i].s_desc);
      strcat(output_string, aux_string);
    }
  }
  if(output_string[0]){
    printf(TITLE_STRING, "Your inventory");
    printf("%s", output_string);
  } else { printf("\n You are not carrying anything.");}
};

u_int8_t verb_hash(){
  if(strlen(verb)==0){return NO_VERB;}
  if(strcmp(verb, "north") == 0 || strcmp(verb, "n") == 0){return 0;}
  if(strcmp(verb, "east") == 0 || strcmp(verb, "e") == 0){return 1;}
  if(strcmp(verb, "south") == 0 || strcmp(verb, "s") == 0){return 2;}
  if(strcmp(verb, "west") == 0 || strcmp(verb, "w") == 0){return 3;}
  if(strcmp(verb, "up") == 0 || strcmp(verb, "u") == 0){return 4;}
  if(strcmp(verb, "down") == 0 || strcmp(verb, "d") == 0){return 5;}
  if(strcmp(verb, "drop") == 0 || strcmp(verb, "D") == 0){return 6;}
  if(strcmp(verb, "examine") == 0 ||  strcmp(verb, "x") == 0 || strcmp(verb, "exam")  == 0 || strcmp(verb, "read")  == 0){return 7;}
  if(strcmp(verb, "help") == 0 || strcmp(verb, "h") == 0){return 8;}
  if(strcmp(verb, "inventory") == 0 || strcmp(verb, "i") == 0){return 9;}
  if(strcmp(verb, "look") == 0 || strcmp(verb, "l") == 0){return 10;}
  if(strcmp(verb, "get") == 0 || strcmp(verb, "take") == 0 || strcmp(verb, "g") == 0){return 11;}
  if(strcmp(verb, "quit") == 0 || strcmp(verb, "q") == 0){return 12;}
  return NOT_A_VERB;
}

void show_help(){
  printf(TITLE_STRING,"Instruccions");
  printf("This is an interactive fiction text game. This is how it works. The computer\n"
           "prints where you are and what you can see, and gives you a prompt: '>'. Then\n"
           "you type whatever you want. Like in 'north', or 'get stone'. Bear in mind\n"
           "the computer will only understand very simple sentences, of mostly one verb\n"
           "and two nouns.\n");
  printf("\nList of Verbs:");
  printf("\n--------------\n");
  printf(VERB_HELP_STRING,"north", "n", "go north");
  printf(VERB_HELP_STRING,"east", "e", "go east");
  printf(VERB_HELP_STRING,"south", "s", "go south");
  printf(VERB_HELP_STRING,"west", "w", "go west");
  printf(VERB_HELP_STRING,"up", "u", "go up");
  printf(VERB_HELP_STRING,"down", "d", "go down");
  printf(VERB_HELP_STRING,"drop [thing]", "r", "drop something, leave something");
  printf(VERB_HELP_STRING,"get [thing]", "g", "get something, placing it in your inventory");  
  printf(VERB_HELP_STRING,"inventory", "i", "show inventory");
  printf(VERB_HELP_STRING,"examine [thing]", "x", "examine something");
  printf(VERB_HELP_STRING,"look", "l", "look around, to examine some specific thing use examine");
  printf(VERB_HELP_STRING,"help", "h", "show this help");
  printf(VERB_HELP_STRING,"quit", "q", "quit game");
}

void parse_player_input(){
  // make it return 1 if changed room, 0 if not
  get_player_input();
  switch(verb_hash()){
    case 0: { if(go(NORTH) == 0){parse_player_input();} break;}
    case 1: { if(go( EAST) == 0){parse_player_input();} break;}
    case 2: { if(go(SOUTH) == 0){parse_player_input();} break;}
    case 3: { if(go( WEST) == 0){parse_player_input();} break;}
    case 4: { if(go(   UP) == 0){parse_player_input();} break;}
    case 5: { if(go( DOWN) == 0){parse_player_input();} break;}
    case 6: {
      u_int8_t dropped = drop_something(first_word);
      if(dropped==0){
        char buffer[(MAX_WORD_SIZE*2)+2] = "";
        strcpy(buffer, first_word);
        strcat(strcat(buffer, " "), second_word);
        drop_something(buffer);
      }
      break;
    }
    case 7: 
    {
      if(strcmp(first_word, "self")==0){
        printf("\n%s\n", CHARACTER_DESCRIPTION);
      } else {
        u_int8_t found_examinandum= examine_something(first_word); 
        if(found_examinandum==0){
          char buffer[(MAX_WORD_SIZE*2)+2];
          // initialize string buffer
          strcpy(buffer, first_word);
          strcat(strcat(buffer, " "), second_word);
          found_examinandum = examine_something(buffer);
          if(found_examinandum==0){printf(ERROR_STRING,"I can't find that");}
        }
        
      }
      break;
    }
    case 8: { show_help(); break;}
    case 9: show_inventory(); break;
    case 10: show_room(&rooms[actual_room_index], 1); break;
    case 11: { 
      u_int8_t got_something = get_something(first_word);
      if(got_something==0){
        char buffer[(MAX_WORD_SIZE*2)+2];
        strcpy(buffer, first_word);
        strcat(strcat(buffer, " "), second_word);
        get_something(buffer);
      }
      
      break;}
    case 12: {
      printf(TITLE_STRING, "Game Over");
      printf("Goodbye! :)");
      game_on = 0;
      break;
    }
    case NO_VERB: 
      {
        printf(ERROR_STRING, "I would need a command.");
        parse_player_input();
        break;}
    default: 
      { 
        printf(ERROR_STRING, "I could not recognize your command.");
        parse_player_input();
      }
  }   
}
// Other functions
void game_start();



int main(){   
  char alfa[150] = "";
  game_on = 1;

  for(int i=0;i<120;i++){
  things[i].l_desc="default thing l_desc";
  things[i].s_desc="default thing s_desc";
  things[i].location = NOWHERE;
} // things default values

  things[0].s_desc =  "book"; 
  things[0].l_desc = 
  "  A black book, with all its pages torn, but one. Its printed letters, in some\n"
  "foreign script make little sense to you. But on those, written with wax,\n" 
  "crayons, there are some much more familiar ones:\n\n"
  "Ding, dong, knell,\n"
  "Kid’s in the cell.\n\n"
  "Who did so bad?\n"
  "The mean old hag.\n\n"
  "She's making me write,\n"
  "She'll see you cry.\n\n"
  "Your fears and tears,\n"
  "Are her best beer.\n\n"
  "T'is a spell,\n"
  "There's no escape,\n\n"  
  "But if you return,\n"
  "My long lost head.\n" ;
  //------------------------------------------------------------------------------
  things[0].location = 0;
  things[0].fixed = 1;

  things[1].s_desc =  "cobweb"; 
  things[1].l_desc = "Exquisite webs of silk and dust where trapped bugs wait for the fang of the weaver";
  things[1].location = 0;
  things[1].fixed = IS_DECORATION;

  things[2].s_desc =  "knife"; 
  things[2].l_desc = "A small, dull knife with a wooden handle. An inscription on the blade reads \"I'm not your friend\" ";
  things[2].location = 0;
  things[2].fixed = 0;
  
  rooms[0] = new_room(
    "Vestibule", 
    "A small room, filled with dust and cobwebs. Nothing unusual or unexpected but\n"
    "for the main door and the window have been replaced by bricks.", 
    1,NOWHERE,NOWHERE,2,NOWHERE,NOWHERE);

  rooms[1] = new_room(
    "Downstairs Hall", 
    "A long empty corridor.",
    3,4,0,5,6,NOWHERE);  


  rooms[2] = new_room(
    "Toilet",
    "A basin, dirty with rust and a flush toilet, thankfully dry.",
    NOWHERE,0,NOWHERE,NOWHERE,NOWHERE,NOWHERE);

  rooms[3] = new_room(
    "Nook",
    "Behind an iron door, scarcely the space for a narrow, thin mattress too short\n"
    "for an adult. A confused mess of letters are scribbled on the wall.",
    NOWHERE,11,1,NOWHERE,NOWHERE,NOWHERE);
  
  things[3].s_desc =  "wall"; 
  things[3].l_desc = "ja ja zhubniguraz\n\tja ja zhubniguraz!\n repaj\noz ur dez shant rest\n\tdeir heds ja ja zhubniguraz ";
  things[3].location = 3;
  things[3].fixed = IS_DECORATION;

  rooms[4] = new_room(
    //------------------------------------------------------------------------------
    "Dining room",
    "The remains of a dozen chairs and a table lay on the dusty ground. There are \n"
    "smithereens of plates and glasses everywhere. Most disturbing:a red ax and a\n"
    "trail of old stains that leads to the north." ,
    11,NOWHERE,NOWHERE,1,NOWHERE,NOWHERE);

  things[4].s_desc =  "chairs"; 
  things[4].l_desc = "pieces of chopped wood, now woodwoorm fodder";
  things[4].location = 4;
  things[4].fixed = IS_DECORATION;

  rooms[5] = new_room(
    //------------------------------------------------------------------------------
    "Living room",
    "Nothing but dust and the fireplace.",
    NOWHERE,NOWHERE,NOWHERE,1,NOWHERE,NOWHERE);

  things[5].s_desc =  "fireplace"; 
  things[5].l_desc =  "When you look closely, you spot a headless figure drawn on the ashes.";
  things[5].location = 5;
  things[5].fixed = IS_DECORATION;

  rooms[6] = new_room(
    //------------------------------------------------------------------------------
    "Upstairs hall",
    "Leading to four smashed-open doors and downstairs.",
    9,10,7,8,NOWHERE,1);

  rooms[7] = new_room(
    //------------------------------------------------------------------------------
    "Children chamber",
    "Two beds, both empty but for the dust.",
    6,NOWHERE,NOWHERE,NOWHERE,NOWHERE,NOWHERE);

  rooms[8] = new_room(
    //------------------------------------------------------------------------------
    "Single bed chamber",
    "An odd setting, the bed is in the middle of the room, \"walled\" by an array of\n"
    "crumbling old furniture. ",
    NOWHERE,6,NOWHERE,NOWHERE,NOWHERE,NOWHERE);

  rooms[9] = new_room(
    //------------------------------------------------------------------------------
    "Bathroom",
    "A basin, a flush toilet, a bidet and bathtub with a shower.",
    NOWHERE,NOWHERE,6,NOWHERE,NOWHERE,NOWHERE);
  
  rooms[10] = new_room(
    //------------------------------------------------------------------------------
    "Twin bed chamber",
    "A tombstone lies on a queen-sized bed.",
    NOWHERE,NOWHERE,NOWHERE,7,NOWHERE,NOWHERE);

  rooms[11] = new_room(
    //------------------------------------------------------------------------------
    "Kitchen",
    "Knives and utensils, all gone. The big kitchen table, turned upside down on the\n"
    "dirty floor. The old coal stove, covered in a one inch layer of black flies\n" 
    "corpses. On the wall, an old rotary telephone. Oddly enough, the pantry is\n"
    "neatly closed. \n",
    NOWHERE,NOWHERE,NOWHERE,NOWHERE,NOWHERE,12);

  rooms[12] = new_room(
    //------------------------------------------------------------------------------
    "Basement",
    ".",
    NOWHERE,NOWHERE,NOWHERE,NOWHERE,11,NOWHERE);
  game_start();
  while(game_on){
    if(went_to_a_new_room){SHOW_ACTUAL_ROOM;}
    went_to_a_new_room = 0;
    parse_player_input();
  }
  return 0;
}

void game_start(){
  printf("\n\n\nDo you want instruccions? (y/N) ");
  get_player_input();
  if(verb[0]=='y'){
    show_help();
  }
  strcpy(verb, "");
  printf("\nDo you want to read the introduction? (y/N) ");
  get_player_input();
  if(verb[0]=='y') {
    printf(TITLE_STRING, "Halloween Ball");
    printf("Halloween, a Saturday. No school, so you just went with your friends for a\n"
        "ball game. An easy and happy morning, no worries. Except Daniel just had\n"
        "this bright idea of playing at the Wet Crow garden. Yes, that fenced waste\n" 
        "where nobody, really nobody, would go. Because it's so creepy.\n\n"
        "\tBut you all went, and everything was alrighty, except for the dead chicks\n"
        "and the thorny bushes, oh and that whizzling wind. Until someone, you, had to\n"
        "kick the ball over the fence and into that house, that huge house where nobody\n"
        "had lived in a century: the local ghost house. And so you boldly went in.\n");
    printf("\n\t\t");
    for(uint8_t i=1; i<60; i++)
      printf("-");
    printf("\n");
  }   
  printf("\nPress enter to continue");
  getchar();
}
