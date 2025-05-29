#include <assert.h>
#include "compiler.h"

int E(); // Declare function E
void STMT(); // Declare function STMT
void IF(); // Declare function IF
void BLOCK(); // Declare function BLOCK

int tempIdx = 0, labelIdx = 0; // Declare global variables tempIdx and labelIdx, used for generating temporary variable indexes and label indexes

#define nextTemp() (tempIdx++) // Define macro nextTemp() to generate the next temporary variable index
#define nextLabel() (labelIdx++) // Define macro nextLabel() to generate the next label index
#define emit printf // Define macro emit as printf function

int isNext(char *set) { // Define function isNext to check if the next token is in the specified set
  char eset[SMAX], etoken[SMAX]; // Declare string variables eset and etoken
  sprintf(eset, " %s ", set); // Format the set into the string eset
  sprintf(etoken, " %s ", tokens[tokenIdx]); // Format the next token into the string etoken
  return (tokenIdx < tokenTop && strstr(eset, etoken) != NULL); // Check if the next token is in the set
}

int isEnd() { // Define function isEnd to check if the token stream has reached the end
  return tokenIdx >= tokenTop; // Return whether the token stream has reached the end
}

char *next() { // Define function next to get the next token
  // printf("token[%d]=%s\n", tokenIdx, tokens[tokenIdx]);
  return tokens[tokenIdx++]; // Return the next token and increment the token index
}

char *skip(char *set) { // Define function skip to skip the next token if it matches the specified set
  if (isNext(set)) { // If the next token is in the specified set
    return next(); // Return the next token
  } else { // Otherwise
    printf("skip(%s) got %s fail!\n", set, next()); // Print an error message and terminate the program
    assert(0);
  }
}

// F = (E) | Number | Id
int F() { // Define function F to parse an expression
  int f; // Declare variable f to store the result
  if (isNext("(")) { // If the next token is a left parenthesis '('
    next(); // Get the next token
    f = E(); // Call function E and assign the result to f
    next(); // Get the next token
  } else { // Otherwise (it is a number or identifier)
    f = nextTemp(); // Get the next temporary variable index and assign it to f
    char *item = next(); // Get the next token and assign it to item
    if (isdigit(*item)) { // If the token is a number
      emit("t%d = %s\n", f, item); // Print the instruction
    } else { // Otherwise (it is an identifier)
      if (isNext("++")) { // If the next token is '++'
        next(); // Get the next token
        emit("%s = %s + 1\n", item, item); // Print the instruction
      }
      emit("t%d = %s\n", f, item); // Print the instruction
    }
  }
  return f; // Return the result
}

// E = F (op E)*
int E() { // Define function E to parse an expression
  int i1 = F(); // Call function F and assign the result to i1
  while (isNext("+ - * / & | ! < > = <= >= == !=")) { // While the next token is an operator
    char *op = next(); // Get the operator
    int i2 = E(); // Call function E and assign the result to i2
    int i = nextTemp(); // Get the next temporary variable index
    emit("t%d = t%d %s t%d\n", i, i1, op, i2); // Print the instruction
    i1 = i; // Update i1
  }
  return i1; // Return the result
}

// FOR = for (ASSIGN EXP; EXP) STMT

// ASSIGN = id '=' E;
void ASSIGN() { // Define function ASSIGN to parse an assignment statement
  char *id = next(); // Get the identifier
  skip("="); // Skip '='
  int e = E(); // Call function E and assign the result to e
  skip(";"); // Skip ';'
  emit("%s = t%d\n", id, e); // Print the instruction
}

// WHILE = while (E) STMT
void WHILE() { // Define function WHILE to parse a while loop
  int whileBegin = nextLabel(); // Get the next label index and assign it to whileBegin
  int whileEnd = nextLabel(); // Get the next label index
  emit("(L%d)\n", whileBegin); // Print the label
  skip("while"); // Skip the 'while' keyword
  skip("("); // Skip '('
  int e = E(); // Call function E and assign the result to e
  emit("if not T%d goto L%d\n", e, whileEnd); // Print the conditional jump instruction
  skip(")"); // Skip ')'
  STMT(); // Call function STMT to parse the statement
  emit("goto L%d\n", whileBegin); // Print the unconditional jump instruction
  emit("(L%d)\n", whileEnd); // Print the label
}

// if (EXP) STMT (else STMT)?
void IF() { // Define function IF to parse an if statement
  skip("if"); // Skip the 'if' keyword
  skip("("); // Skip '('
  E(); // Call function E to parse the expression
  skip(")"); // Skip ')'
  STMT(); // Call function STMT to parse the statement
  if (isNext("else")) { // If the next token is 'else'
    skip("else"); // Skip the 'else' keyword
    STMT(); // Call function STMT to parse the statement
  }
}

// DOWHILE = do STMT while (E)
void DOWHILE() { // Define function DOWHILE to parse a do-while loop
  int dowhileBegin = nextLabel(); // Get the next label index and assign it to dowhileBegin
  skip("do"); // Skip the 'do' keyword
  emit("(L%d)\n", dowhileBegin); // Print the label
  STMT(); // Call function STMT to parse the statement
  skip("while"); // Skip the 'while' keyword
  skip("("); // Skip '('
  int e = E(); // Call function E to parse the expression
  emit("if T%d goto L%d\n", e, dowhileBegin); // Print the conditional jump instruction
  skip(")"); // Skip ')'
  skip(";"); // Skip ';'
}

// STMT = WHILE | BLOCK | ASSIGN
void STMT() { // Define function STMT to parse a statement
  if (isNext("while")) // If the next token is 'while'
    return WHILE(); // Call function WHILE to parse the loop
  else if(isNext("do")) // If the next token is 'do'
    return DOWHILE(); // Call function DOWHILE to parse the loop
  else if (isNext("if")) // If the next token is 'if'
    IF(); // Call function IF to parse the conditional statement
  else if (isNext("{")) // If the next token is '{'
    BLOCK(); // Call function BLOCK to parse the block
  else
    ASSIGN(); // Otherwise, call function ASSIGN to parse the assignment statement
}

// STMTS = STMT*
void STMTS() { // Define function STMTS to parse a series of statements
  while (!isEnd() && !isNext("}")) { // While the token stream has not ended and the next token is not '}'
    STMT(); // Call function STMT to parse a statement
  }
}

// BLOCK = { STMTS }
void BLOCK() { // Define function BLOCK to parse a block
  skip("{"); // Skip '{'
  STMTS(); // Call function STMTS to parse a series of statements
  skip("}"); // Skip '}'
}

// PROG = STMTS
void PROG() { // Define function PROG to parse the entire program
  STMTS(); // Call function STMTS to parse a series of statements
}

void parse() { // Define function parse to start parsing the program
  printf("============ parse =============\n"); // Print a message
  tokenIdx = 0; // Initialize token index
  PROG(); // Call function PROG to parse the entire program
}
