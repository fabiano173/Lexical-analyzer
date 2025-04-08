#include <iostream>
#include <string>
#include <cctype>
#include <sstream>

using namespace std;
enum CharClass {
    LETTER,
    DIGIT,
    UNKNOWN,
    END_OF_FILE
};
enum Token {
    INT_LIT = 10,
    IDENT = 11,
    ASSIGN_OP = 20,
    ADD_OP = 21,
    SUB_OP = 22,
    MULT_OP = 23,
    DIV_OP = 24,
    LEFT_PAREN = 25,
    RIGHT_PAREN = 26,
    END_OF_INPUT = -1
};
class Lexer {
private:
    string input;
    size_t pos;
    CharClass char_class;
    string lexeme;
    char next_char;
    Token next_token;

    void addChar() {
        lexeme += next_char;
    }

    void getChar() {
        if (pos < input.length()) {
            next_char = input[pos++];
            if (isalpha(next_char)) {
                char_class = LETTER;
            }
            else if (isdigit(next_char)) {
                char_class = DIGIT;
            }
            else {
                char_class = UNKNOWN;
            }
        }
        else {
            char_class = END_OF_FILE;
        }
    }

    void getNonBlank() {
        while (pos < input.length() && isspace(next_char)) {
            getChar();
        }
    }

    Token lookup(char ch) {
        switch (ch) {
        case '(':
            addChar();
            return LEFT_PAREN;
        case ')':
            addChar();
            return RIGHT_PAREN;
        case '+':
            addChar();
            return ADD_OP;
        case '-':
            addChar();
            return SUB_OP;
        case '*':
            addChar();
            return MULT_OP;
        case '/':
            addChar();
            return DIV_OP;
        case '=':
            addChar();
            return ASSIGN_OP;
        default:
            addChar();
            return END_OF_INPUT;
        }
    }

public:
    Lexer(const string& expression) : input(expression), pos(0) {
        getChar();
    }

    Token lex() {
        lexeme.clear();
        getNonBlank();

        switch (char_class) {
        case LETTER:
            addChar();
            getChar();
            while (char_class == LETTER || char_class == DIGIT) {
                addChar();
                getChar();
            }
            next_token = IDENT;
            break;

        case DIGIT:
            addChar();
            getChar();
            while (char_class == DIGIT) {
                addChar();
                getChar();
            }
            next_token = INT_LIT;
            break;

        case UNKNOWN:
            next_token = lookup(next_char);
            getChar();
            break;

        case END_OF_FILE:
            next_token = END_OF_INPUT;
            lexeme = "EOF";
            break;
        }

        cout << "Next token is: " << next_token
            << ", Next lexeme is " << lexeme << endl;
        return next_token;
    }

    void analyze() {
        cout << "Analyzing expression: " << input << endl;
        do {
            lex();
        } while (next_token != END_OF_INPUT);
    }
};

int main() {
    string user_input;

    cout << "Simple Arithmetic Expression Lexer" << endl;
    cout << "Supported tokens: identifiers, integers, +, -, *, /, (, ), =" << endl;
    cout << "Enter an expression (or 'quit' to exit):" << endl;

    while (true) {
        cout << "> ";
        getline(cin, user_input);

        if (user_input == "quit") {
            break;
        }

        if (!user_input.empty()) {
            Lexer lexer(user_input);
            lexer.analyze();
        }
    }

    cout << "Lexer session ended." << endl;
    return 0;
}
