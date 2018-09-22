#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <ctype.h>
#include <sys/types.h>
#include <sys/stat.h>
#include <unistd.h>
#include <errno.h>
#include <dirent.h>

// function prototypes
void parse_line(char *command);
char **tokenize(char *line, int *num_tokens);

// static variables
static char *line;
static char *name;
static int last_status;
static char* last_command;

// main function
int main(void) {
	name = (char*)malloc(256*sizeof(char));
	strcpy(name, "mysh");
	last_status = 0;
	// last_command = (char*)malloc(256*sizeof(char));

	line = (char*)malloc(256*sizeof(char));
	// main loop used for interactive mode

	if(isatty(0)) {
		
		while(1)
		{
			printf("%s> ", name);
			if(!fgets(line, 256, stdin)){
				exit(0);
			}
			parse_line(line);
		}
	}
	else 
	{
		while(fgets(line, 256, stdin))
		{
			parse_line(line);
		}
	}	
	exit(0);

}

void parse_line(char *line)
{

	int *num_tokens = (int*)malloc(sizeof(int));
	*num_tokens = 0;

	char** tokens = tokenize(line, num_tokens);

	if(tokens[0][0] == '#') {
		return;
	}

	if(!strcmp(tokens[0], "name"))
	{
		if(*num_tokens == 1) {
			printf("%s\n", name);
		} else {
			name = tokens[1];
			last_status = 0;	
		}
	}

	if(!strcmp(tokens[0], "exit"))
	{
		int status = atoi(tokens[1]);
		exit(status);
	}

	if(!strcmp(tokens[0], "print"))
	{
		for(int i = 1; i < *num_tokens; i++) {
			printf((i == *num_tokens - 1) ? "%s" : "%s ", tokens[i]);
		}
		last_status = 0;
	}

	if(!strcmp(tokens[0], "echo"))
	{
		for(int i = 1; i < *num_tokens; i++) {
			printf("%s ", tokens[i]);
		}
		printf("\n");
		last_status = 0;
	}

	if(!strcmp(tokens[0], "pid"))
	{
		pid_t pid = getpid();
		printf("%d\n", pid);
		last_status = 0;
	}

	if(!strcmp(tokens[0], "ppid"))
	{
		pid_t ppid = getppid();
		printf("%d\n", ppid);
		last_status = 0;
	}

	if(!strcmp(tokens[0], "help"))
	{
		printf("TODO\n");
		last_status = 0;
	}

	if(!strcmp(tokens[0], "status"))
	{
		if(last_status != 0) {
			printf("%s: %s\n", last_command, strerror(last_status));	
		}
		printf("%d\n", last_status);
		last_status = 0;
	}

	if(!strcmp(tokens[0], "dirchange"))
	{
		char *dir_path;
		dir_path = (*num_tokens == 1) ? "/" : tokens[1];

		if((last_status = chdir(dir_path)) < 0)
		{
			last_status = errno;
		}
	}

	if(!strcmp(tokens[0], "dirwhere"))
	{
		char *cwd = (char*)malloc(256*sizeof(char));

		if((getcwd(cwd, 256)) < 0)
		{
			last_status = errno;
			return;
		} else {
			last_status = 0;
			printf("%s\n", cwd);
		}
	}

	if(!strcmp(tokens[0], "dirmake"))
	{
		if((last_status = mkdir(tokens[1], 0700)) < 0)
		{
			last_status = errno;
			last_command = "dirmake";
		}
	}

	if(!strcmp(tokens[0], "dirremove"))
	{
		if((last_status = rmdir(tokens[1])) < 0)
		{
			last_status = errno;
		}
	}

	if(!strcmp(tokens[0], "dirlist"))
	{

		struct dirent *entry;
		DIR *dir = opendir((*num_tokens == 1) ? "." : tokens[1]);

		if(dir == NULL) {
			last_status = errno;
			return;
		} else {
			last_status = 0;
		}

		while((entry = readdir(dir)) != NULL)
		{
            printf("%s  ", entry->d_name);
		}
		printf("\n");
		last_status = 0;
	}

	// Make hard link to destination.
	if(!strcmp(tokens[0], "linkhard"))
	{
		if((last_status = link(tokens[1], tokens[2])) < 0) {
			last_status = errno;
		}
	}

	if(!strcmp(tokens[0], "linksoft"))
	{
		if((last_status = symlink(tokens[1], tokens[2])) < 0) {
			last_status = errno;
		}
	}

	if(!strcmp(tokens[0], "linkread"))
	{
		char *buff = malloc(256*sizeof(char*));
		if(readlink(tokens[1], buff, 256) < 0) {
			last_status = errno;
			return;
		} else {
			last_status = 0;
		}
		printf("%s\n", buff);
	}

	if(!strcmp(tokens[0], "linklist"))
	{
		struct dirent *entry;
		DIR *dir = opendir(".");

		if(dir == NULL) {
			last_status = errno;
			return;
		} else {
			last_status = 0;
		}

		struct stat f;
		
		int ref_inode;
		stat(tokens[1], &f);
		ref_inode = f.st_ino;

		while((entry = readdir(dir)) != NULL)
		{
			lstat(entry->d_name, &f);
			if(f.st_ino == ref_inode)
			{
				printf("%s  ", entry->d_name);
			}

		}
		printf("\n");

		last_status = 0;

	}

	if(!strcmp(tokens[0], "unlink"))
	{
		if((last_status = unlink(tokens[1])) < 0) {
			last_status = errno;
		}
	}

	if(!strcmp(tokens[0], "rename"))
	{
		if((last_status = rename(tokens[1], tokens[2])) < 0) {
			last_status = errno;
		}
	}

	if(!strcmp(tokens[0], "cpcat"))
	{

	}
}

char **tokenize(char *line, int *num_tokens)
{
	char *buff = (char*)malloc(256*sizeof(char));
	char **res = (char**)malloc(256*sizeof(char*));

	int buff_index = 0;
	int res_index = 0;

	for(int i = 0; i < strlen(line) - 1; i++)
	{
		if(isspace(line[i]))
		{
			buff[buff_index] = '\0';
			res[res_index] = malloc((strlen(buff) + 1)*sizeof(char*));
			strcpy(res[res_index], buff);
			res_index++;
			buff_index = 0;
			(*num_tokens)++;

			while(line[i] == ' ') {
				i++;
			}
			i--;
		}
		else if(line[i] == '"')
		{
			i++;
			while(line[i] != '"')
			{
				buff[buff_index] = line[i];
				buff_index++;
				i++;
			}
		}
		else
		{
			buff[buff_index] = line[i];
			buff_index++;
		}
	}


	buff[buff_index] = '\0';
	res[res_index] = malloc((strlen(buff) + 1)*sizeof(char*));
	strcpy(res[res_index], buff);
	res_index++;
	(*num_tokens)++;

	return res;
}