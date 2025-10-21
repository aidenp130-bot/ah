#include <inttypes.h>
#include <stdbool.h>
#include <stdint.h>
#include <stdio.h>
#include <stdlib.h>
#include <string.h>

const char *FLAG_1 = "osu_flag1.txt";
const char *FLAG_2 = "osu_flag2.txt";
const char *FLAG_3 = "osu_flag3.txt";

const char NORMAL = 0;
const char TESTER = 1;
const char ADMIN = 78;

typedef struct __attribute__((__packed__)) User {
  int32_t skill;
  char name[24];
  char type;
  char title[8];
} User;

typedef struct __attribute__((__packed__)) Work {
  int32_t numbers_count;
  char friend[4];
  int32_t numbers[8];
  char message[64];
} Work;

void read_file(const char *filename, char *buffer, int32_t size) {
  FILE *file = fopen(filename, "r");

  int32_t str_len = size - 1;
  if (file) {
    size_t read_size = fread(buffer, sizeof(char), str_len, file);
    buffer[read_size] = '\0';
  } else {
    strncpy(buffer, filename, str_len);
    buffer[str_len] = '\0';
  }
}

void print_flag(const char *filename) {
  char buffer[64];
  read_file(filename, buffer, 64);

  printf("%s", buffer);
}

void seek_newline() {
  int c;
  while ((c = getchar()) != '\n') {
    if (c == EOF) {
      exit(1);
    }
  }
}

void read_input(char *buf, int32_t buf_size) {
  fflush(stdout);

  if (!fgets(buf, buf_size, stdin)) {
    exit(1);
  }

  int32_t last_char_index = strlen(buf) - 1;
  if (last_char_index >= 0 && buf[last_char_index] == '\n') {
    buf[last_char_index] = '\0';
  } else {
    seek_newline();
  }
}

int32_t get_number() {
  char buffer[16];
  read_input(buffer, 16);

  return atoi(buffer);
}

void get_user(User *user) {
  user->skill = 0;
  user->type = NORMAL;

  printf("Title: ");
  read_input(user->title, 8);

  printf("Name: ");
  read_input(user->name, 32);

  char buffer[16];
  printf("Special user login (blank for nothing): ");
  read_input(buffer, 16);

  if (strcmp(buffer, "strong_password123") == 0) {
    user->type = ADMIN;
  }

  if (strcmp(buffer, "tester_0213") == 0) {
    user->type = TESTER;
  }

  printf("\n");
}

void print_user(User *user) {
  if (user->type == NORMAL) {
    printf("USER ");
  } else if (user->type == ADMIN) {
    printf("ADMIN ");
  } else {
    printf("TESTER ");
  }

  printf("%s %s", user->title, user->name);
}

void introduce_user(User *user) {
  printf("Hello\n");
  print_user(user);
  printf("\n\n");

  if (user->type == ADMIN) {
    print_flag(FLAG_1);
    printf("\n\n");
  }

  if (user->type == TESTER) {
    printf("Changelog:\n");
    printf("\tPrint more newlines\n");
    printf("\tDecreased max user name length\n");
    printf("\tAdded friend\n");
  }

  printf("Welcome to your custom work station\n");
}

int32_t add_numbers(Work *work) {
  int32_t number = 0;
  for (int32_t i = 0; i < work->numbers_count; i++) {
    number += work->numbers[i];
  }

  return number;
}

int main() {
  User user;
  get_user(&user);
  introduce_user(&user);
  printf("\n");

  bool done = false;
  Work work =
      (Work){.numbers_count = 0, .friend = ":)", .message = "Hello World!"};
  while (!done) {
    print_user(&user);
    printf("\nWhat would you like to do?\n");
    printf("\t0: Nothing\n");
    printf("\t1: Add number\n");
    printf("\t2: Remove a number\n");
    printf("\t3: Check on your friend\n");
    printf("\t4: Change your message\n");
    printf("\t5: Read your message\n");
    printf("\t6: Exit\n\n");

    uint32_t choice = get_number();
    printf("\n");
    switch (choice) {
    case 1:
      if (work.numbers_count < 8) {
        printf("Number: ");
        work.numbers[work.numbers_count] = get_number();
        work.numbers_count++;
        // C out concatenates strings that are next to each other eg "ab" "bc"
        // == "abbc" PRId32 is a string that will print a int32_t so either d or
        // l
        printf("Your total is %" PRId32 "\n\n", add_numbers(&work));
      } else {
        printf("Too many numbers\n\n");
      }

      break;
    case 2:
      if (work.numbers_count >= 0) {
        work.numbers_count--;
        printf("Your total is %" PRId32 "\n\n", add_numbers(&work));
      } else {
        printf("No numbers left\n\n");
      }

      break;
    case 3:
      printf("He's doing okay right?\n");
      if (work.friend[0] == ':' && work.friend[1] == ')') {
        printf("Yes he is right here: %s\n\n", work.friend);
      } else if (work.friend[0] == 'X' && work.friend[1] == 'P') {
        printf("You killed him! The ultimate red ");
        print_flag(FLAG_2);
        printf(".\n\n");
      } else {
        printf("No he's gone!\n\n");
      }

      break;
    case 4:
      printf("What message: ");
      read_input(work.message, 64);

      if (strcmp(work.message, "flag please ... PLEASE") == 0) {
        printf("Fine.\n\n");
        read_file(FLAG_3, work.message, 64);
      } else {
        printf("\n");
      }

      break;
    case 5:
      if (work.message[0] == 'o' && work.message[1] == 's' &&
          work.message[2] == 'u') {
        printf("SECURE MESSAGE DETECTED\n");
        print_user(&user);
        printf(
            " is not in the sudoers file. This incident will be reported.\n\n");
      } else {
        printf("%s\n\n", work.message);
      }

      break;
    case 6:
      done = true;
      break;
    }
  }

  return 0;
}
