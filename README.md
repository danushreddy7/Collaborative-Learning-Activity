# Collaborative-Learning-Activity:
# NAME:G P Hariesh
# REG NO:212224040100

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
#include <stdlib.h>
#include <string.h>
%}

%%

[ \t\n]    ; /* Skip all whitespace including newlines */

"1"        { return MENU1; }
"2"        { return MENU2; }
"3"        { return MENU3; }

"O"        { yylval.grade = strdup("O"); return GRADE; }
"A+"       { yylval.grade = strdup("A+"); return GRADE; }
"A"        { yylval.grade = strdup("A"); return GRADE; }
"B+"       { yylval.grade = strdup("B+"); return GRADE; }
"B"        { yylval.grade = strdup("B"); return GRADE; }
"C"        { yylval.grade = strdup("C"); return GRADE; }
"P"        { yylval.grade = strdup("P"); return GRADE; }
"F"        { yylval.grade = strdup("F"); return GRADE; }

[0-9]+     { yylval.credit = atoi(yytext); return CREDIT; }

.          { 
              fprintf(stderr, "Invalid character: %s\n", yytext); 
              return ERROR; 
          }

%%

int yywrap() {
    return 1;
}

```

# cgpa.y:
```
%{
#include <stdio.h>
#include <stdlib.h>
#include <string.h>

typedef struct {
    char *grade;
    int credit;
} Subject;

typedef struct {
    Subject *subjects;
    int count;
    int capacity;
    double gpa;
} Semester;

Semester *semesters = NULL;
int current_semester = 0;
int semester_count = 0;
int semester_capacity = 0;

void add_semester();
void add_subject_to_current_semester(char *grade, int credit);
void compute_current_semester_gpa();
void compute_final_cgpa();
double grade_to_point(char *grade);
void free_memory();

int yylex();
void yyerror(const char *s);
%}

%union {
    char *grade;
    int credit;
}

%token <grade> GRADE
%token <credit> CREDIT
%token MENU1 MENU2 MENU3
%token ERROR

%%

input: 
    | input line
    ;

line:
      MENU1 GRADE CREDIT
      {
          if (semester_count == 0) {
              add_semester();
          }
          add_subject_to_current_semester($2, $3);
          printf("Added subject: %s %d\n", $2, $3);
      }
    | MENU2
      {
          if (semester_count > 0) {
              compute_current_semester_gpa();
          }
          add_semester();
          printf("Moving to next semester...\n");
      }
    | MENU3
      {
          if (semester_count > 0) {
              compute_current_semester_gpa();
              compute_final_cgpa();
          }
          free_memory();
          printf("Program finished.\n");
          exit(0);
      }
    | ERROR
      {
          fprintf(stderr, "Skipping invalid input...\n");
      }
    ;

%%

void add_semester() {
    if (semester_count >= semester_capacity) {
        semester_capacity = semester_capacity == 0 ? 2 : semester_capacity * 2;
        semesters = realloc(semesters, semester_capacity * sizeof(Semester));
        if (!semesters) {
            fprintf(stderr, "Memory allocation failed!\n");
            exit(1);
        }
    }
    
    semesters[semester_count].subjects = NULL;
    semesters[semester_count].count = 0;
    semesters[semester_count].capacity = 0;
    semesters[semester_count].gpa = 0.0;
    
    current_semester = semester_count;
    semester_count++;
    printf("Started Semester %d\n", semester_count);
}

void add_subject_to_current_semester(char *grade, int credit) {
    Semester *current = &semesters[current_semester];
    
    if (current->count >= current->capacity) {
        current->capacity = current->capacity == 0 ? 4 : current->capacity * 2;
        current->subjects = realloc(current->subjects, current->capacity * sizeof(Subject));
        if (!current->subjects) {
            fprintf(stderr, "Memory allocation failed!\n");
            exit(1);
        }
    }
    
    current->subjects[current->count].grade = grade;
    current->subjects[current->count].credit = credit;
    current->count++;
}

double grade_to_point(char *grade) {
    if (strcmp(grade, "O") == 0) return 10.0;
    if (strcmp(grade, "A+") == 0) return 9.0;
    if (strcmp(grade, "A") == 0) return 8.0;
    if (strcmp(grade, "B+") == 0) return 7.0;
    if (strcmp(grade, "B") == 0) return 6.0;
    if (strcmp(grade, "C") == 0) return 5.0;
    if (strcmp(grade, "P") == 0) return 4.0;
    if (strcmp(grade, "F") == 0) return 0.0;
    return 0.0;
}

void compute_current_semester_gpa() {
    Semester *current = &semesters[current_semester];
    double total_points = 0.0;
    int total_credits = 0;
    int i;
    
    if (current->count == 0) {
        printf("No subjects in semester %d\n", current_semester + 1);
        return;
    }
    
    printf("\nComputing GPA for Semester %d:\n", current_semester + 1);
    for (i = 0; i < current->count; i++) {
        double points = grade_to_point(current->subjects[i].grade);
        total_points += points * current->subjects[i].credit;
        total_credits += current->subjects[i].credit;
        printf("  %s %d -> %.1f points\n", 
               current->subjects[i].grade, 
               current->subjects[i].credit, 
               points);
    }
    
    if (total_credits > 0) {
        current->gpa = total_points / total_credits;
        printf("Semester %d GPA = %.2f\n", current_semester + 1, current->gpa);
    }
}

void compute_final_cgpa() {
    double total_weighted_gpa = 0.0;
    int total_credits = 0;
    int i, j;
    
    printf("\nComputing Final CGPA:\n");
    for (i = 0; i < semester_count; i++) {
        int semester_credits = 0;
        for (j = 0; j < semesters[i].count; j++) {
            semester_credits += semesters[i].subjects[j].credit;
        }
        
        total_weighted_gpa += semesters[i].gpa * semester_credits;
        total_credits += semester_credits;
        printf("  Semester %d: GPA=%.2f, Credits=%d\n", 
               i + 1, semesters[i].gpa, semester_credits);
    }
    
    if (total_credits > 0) {
        double cgpa = total_weighted_gpa / total_credits;
        printf("Final CGPA = %.2f\n", cgpa);
    }
}

void free_memory() {
    int i, j;
    for (i = 0; i < semester_count; i++) {
        for (j = 0; j < semesters[i].count; j++) {
            free(semesters[i].subjects[j].grade);
        }
        free(semesters[i].subjects);
    }
    free(semesters);
}

void yyerror(const char *s) {
    fprintf(stderr, "Syntax error: %s\n", s);
    fprintf(stderr, "Expected format: '1 grade credit' or '2' or '3'\n");
    fprintf(stderr, "Valid grades: O, A+, A, B+, B, C, P, F\n");
}

int main() {
    printf("=== CGPA Calculator ===\n");
    printf("Menu Options:\n");
    printf("  1 <grade> <credit>  - Add subject (e.g., 1 A 4)\n");
    printf("  2                   - Next semester\n");
    printf("  3                   - Finish program and compute CGPA\n");
    printf("Valid grades: O, A+, A, B+, B, C, P, F\n");
    printf("\nEnter your choices (one per line):\n");
    
    // Start with first semester
    add_semester();
    
    yyparse();
    return 0;
}
```

## Output
![WhatsApp Image 2025-11-24 at 14 34 56_e1bc872d](https://github.com/user-attachments/assets/a6f7b228-d4f6-464a-9476-8090d8c3499d)


## RESULT:
The CGPA Calculator program was successfully developed using Flex and Bison.

