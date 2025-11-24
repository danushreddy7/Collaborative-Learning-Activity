# Collaborative-Learning-Activity:
# NAME: T.DANUSH REDDY
# REG NO:212223040029

## PROBLEM STATEMENT:

# Create a CGPA calculator using Flex and Bison that allows users to enter grades and credits for multiple semesters using a menu-driven interface and compute GPA and CGPA.
## PROCEDURE:
# STEP-1:Tokens in Flex:
* Recognize the following:- Grades: O, A+, A, B+, B, C, P, F- Numbers (credits)
* Menu options: 1, 2, 3 3. 
# STEP-2:Grammar in Bison: 
* MENU1 → Add subject
* MENU2 → Next semester
* MENU3 → Finish program
* GRADE NUMBER → store grade and credit
# STEP-3: yylval Usage: 
* yylval is used to send values from lexer to parser.
* Example:- yylval.grade for grade strings- yylval.credit for integers 
# STEP-4:grade_to_point() Function: 
* Maps grade strings to numeric grade points
# STEP-5:GPA Formula GPA:
 <img width="753" height="25" alt="image" src="https://github.com/user-attachments/assets/d28399af-167b-4c34-a30d-175eb2f82c3c" />
# STEP-6:Bison Actions:
* MENU1: Add subject record
* MENU2: Compute semester GPA
* MENU3: Compute final CGPA and exit
# STEP-7:Example Input:
1  
A 4  
1  
B+ 3  
2  
1  
O 3 

## program:
# cgpa.l:
```
%{
#include "cgpa.tab.h"
#include <string.h>
%}

%%
[ \t\r\n]+                  ;     // Ignore whitespace

"O"                         { yylval.grade = strdup(yytext); return GRADE; }
"A+"                        { yylval.grade = strdup(yytext); return GRADE; }
"A"                         { yylval.grade = strdup(yytext); return GRADE; }
"B+"                        { yylval.grade = strdup(yytext); return GRADE; }
"B"                         { yylval.grade = strdup(yytext); return GRADE; }
"C"                         { yylval.grade = strdup(yytext); return GRADE; }
"P"                         { yylval.grade = strdup(yytext); return GRADE; }
"F"                         { yylval.grade = strdup(yytext); return GRADE; }

[0-9]+                      { yylval.credit = atoi(yytext); return NUMBER; }

"1"                         { return MENU1; }
"2"                         { return MENU2; }
"3"                         { return MENU3; }

.                           ;     // Ignore unknown characters
%%
```
# cgpa.y:
```
%{
#include <stdio.h>
#include <stdlib.h>
#include <string.h>

int yylex();
void yyerror(const char *s);

float grade_to_point(char *g);

// Semester values
float sem_points = 0, sem_credits = 0;

// CGPA values
float total_points = 0, total_credits = 0;

int semester_no = 1;
%}

%union {
    char *grade;
    int credit;
}

%token <grade> GRADE
%token <credit> NUMBER
%token MENU1 MENU2 MENU3

%%
program:
        program menu_action
      | /* empty */
      ;

menu_action:
        MENU1 GRADE NUMBER
        {
            float gp = grade_to_point($2);
            sem_points += gp * $3;
            sem_credits += $3;
        }

      | MENU2
        {
            float gpa = sem_points / sem_credits;
            printf("Semester %d GPA = %.2f\n", semester_no, gpa);

            total_points += sem_points;
            total_credits += sem_credits;

            semester_no++;

            sem_points = 0;
            sem_credits = 0;
        }

      | MENU3
        {
            float cgpa = total_points / total_credits;
            printf("Final CGPA = %.2f\n", cgpa);
            exit(0);
        }
      ;
%%
void yyerror(const char *s)
{
    printf("Error: %s\n", s);
}

float grade_to_point(char *g)
{
    if (strcmp(g, "O")==0)  return 10;
    if (strcmp(g, "A+")==0) return 9;
    if (strcmp(g, "A")==0)  return 8;
    if (strcmp(g, "B+")==0) return 7;
    if (strcmp(g, "B")==0)  return 6;
    if (strcmp(g, "C")==0)  return 5;
    if (strcmp(g, "P")==0)  return 4;
    return 0; // F
}

int main() {
    printf("=== CGPA Calculator ===\n");
    printf("1. Add subject\n");
    printf("2. Next semester\n");
    printf("3. Finish program\n\n");
    yyparse();
    return 0;
}
```
## OUTPUT:
<img width="1024" height="1536" alt="CD-COLAB" src="https://github.com/user-attachments/assets/29841855-e00c-45e8-a1d8-23aadef45547" />
## RESULT:
The CGPA Calculator program was successfully developed using Flex and Bison.

