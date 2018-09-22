#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <ctype.h>
#include <sys/types.h>
#include <sys/stat.h>
#include <unistd.h>
#include <errno.h>
#include <dirent.h>
#include <sys/wait.h>
#include <fcntl.h>

// function prototypes
void parse_line(char *command);
char **tokenize(char *line, int *num_tokens);
void sigchld_handler();

// static variables
static char *line;
static char *name;
static int last_status;
static char* last_command;

// main function
int main(void) {
	// Set initial shell name.
	name = (char*)malloc(256*sizeof(char));
	strcpy(name, "mysh");
	// Set initial last status.
	last_status = 0;
	// Allocate memory for next line.
	line = (char*)malloc(256*sizeof(char));

	signal(SIGCHLD, sigchld_handler);

	// isatty(0): returns true if in interacive mode
	if(isatty(0)) {
		// Main loop for interactive mode
		while(1)
		{
			// Print prompt
			printf("%s> ", name);
			// Read next line
			if(!fgets(line, 256, stdin)){
				exit(0);
			}
			// Parse and process next line.
			parse_line(line);
		}
	}
	else 
	// loop for evaluating commands in files in non-interactive mode
	{
		// While there are more lines...
		while(fgets(line, 256, stdin))
		{
			// Parse and process next line.
			parse_line(line);
		}
	}
	//wait(NULL);
	exit(0);

}

// parse_line: Parse, tokenize and process the next line entered in the command prompt
void parse_line(char *line)
{

	// If line is a commentor a blank line...
	if(line[0] == '#' || line[0] == '\n') {
		return;
	}
	// Initialize token counter.
	int *num_tokens = (int*)malloc(sizeof(int));
	*num_tokens = 0;

	// Tokenize the next line. num_tokens now contains the number of tokens in the line
	char** tokens = tokenize(line, num_tokens);

	// If line is a comment or an empty line, return.
	if(tokens[0][0] == '#' || tokens[0][0] == '\n') {
		return;
	}

	// Command processing ///////////////////////////////////////////
	int daemon = 0;

	if(!strcmp(tokens[*num_tokens - 1], "&"))
	{
		daemon = 1;
		char **tokens_trunc = (char**)malloc((*num_tokens - 1)*sizeof(char*));
		for(int i = 0; i < *num_tokens - 1; i++)
		{
			tokens_trunc[i] = tokens[i];
		}
		(*num_tokens)--;

		tokens = tokens_trunc;
	}

	// set name
	if(!strcmp(tokens[0], "name"))
	{
		if(*num_tokens == 1) {
			printf("%s\n", name);
		} else {
			name = tokens[1];
			last_status = 0;	
		}
		fflush(stdout);
		return;
	}

	// exit with given status.
	if(!strcmp(tokens[0], "exit"))
	{
		int status = atoi(tokens[1]);
		exit(status);
		return;
	}

	// print arguments without trailing newline
	if(!strcmp(tokens[0], "print"))
	{
		for(int i = 1; i < *num_tokens; i++) {
			printf((i == *num_tokens - 1) ? "%s" : "%s ", tokens[i]);
		}
		fflush(stdout);
		last_status = 0;
		return;
	}

	// print arguments with trailing newline
	if(!strcmp(tokens[0], "echo"))
	{
		for(int i = 1; i < *num_tokens; i++) {
			printf("%s ", tokens[i]);
		}
		printf("\n");
		fflush(stdout);
		last_status = 0;
		return;
	}

	// Print this process' pid
	if(!strcmp(tokens[0], "pid"))
	{
		pid_t pid = getpid();
		printf("%d\n", pid);
		fflush(stdout);
		last_status = 0;
		return;
	}

	// print this process' parent pid
	if(!strcmp(tokens[0], "ppid"))
	{
		pid_t ppid = getppid();
		printf("%d\n", ppid);
		fflush(stdout);
		last_status = 0;
		return;
	}

	// print help
	if(!strcmp(tokens[0], "help"))
	{
		printf("TODO\n");
		fflush(stdout);
		last_status = 0;
		return;
	}

	// print status of last executed command
	if(!strcmp(tokens[0], "status"))
	{
		if(last_status != 0) {
			printf("%s: %s\n", last_command, strerror(last_status));	
		}
		printf("%d\n", last_status);
		fflush(stdout);
		last_status = 0;
		return;
	}

	// change directory
	if(!strcmp(tokens[0], "dirchange"))
	{
		char *dir_path;
		dir_path = (*num_tokens == 1) ? "/" : tokens[1];

		if((last_status = chdir(dir_path)) < 0)
		{
			last_status = errno;
		}
		return;
	}

	// print current working directory
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
			fflush(stdout);
		}
		return;
	}

	// make new directory
	if(!strcmp(tokens[0], "dirmake"))
	{
		if((last_status = mkdir(tokens[1], 0700)) < 0)
		{
			last_status = errno;
			last_command = "dirmake";
		}
		return;
	}

	// remove directory
	if(!strcmp(tokens[0], "dirremove"))
	{
		if((last_status = rmdir(tokens[1])) < 0)
		{
			last_status = errno;
		}

		return;
	}

	// list files in current working directory
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
		fflush(stdout);
		printf("\n");
		last_status = 0;

		return;
	}

	// make hard link to destination
	if(!strcmp(tokens[0], "linkhard"))
	{
		if((last_status = link(tokens[1], tokens[2])) < 0) {
			last_status = errno;
		}
		return;
	}

	// make symbolic link to destination
	if(!strcmp(tokens[0], "linksoft"))
	{
		if((last_status = symlink(tokens[1], tokens[2])) < 0) {
			last_status = errno;
		}
		return;
	}


	// print destination of symbolic link
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
		fflush(stdout);
		return;
	}

	// find all hard links to given file (compare inode number)
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
				printf("%s ", entry->d_name);
			}

		}
		printf("\n");
		fflush(stdout);

		last_status = 0;
		return;
	}

	// delete file
	if(!strcmp(tokens[0], "unlink"))
	{
		if((last_status = unlink(tokens[1])) < 0) {
			last_status = errno;
		}
		return;
	}

	// rename file
	if(!strcmp(tokens[0], "rename"))
	{
		if((last_status = rename(tokens[1], tokens[2])) < 0) {
			last_status = errno;
		}
		return;
	}

	// cpcat: copy and cat
	if(!strcmp(tokens[0], "cpcat"))
	{
		int read_in = -1;
		// If first argument not given or equal to '-', open stdin as input stream.
		if(*num_tokens == 1 || tokens[1][0] == '-') 
		{
			read_in = 0;
		} 
		// Else open specified file.
		else 
		{
			if((read_in = open(tokens[1], O_RDONLY)) < 0)
			{
				last_status = errno;
				return;
			}	
		}

		// open output stream
		int write_out = -1;
		// If output stream argument not given or equal to -, open stdout.
		if(*num_tokens < 3 || tokens[2][0] == '-') 
		{
			write_out = 1;
		}
		// Else, open specified stream.
		else 
		{
			// Check if file exists. If it does not, create.
			if(access(tokens[2], F_OK) == -1)
			{
				creat(tokens[2], S_IRWXU | S_IWUSR | S_IRGRP | S_IROTH);
			}

			// Write to file.
			if((write_out = open(tokens[2], O_WRONLY)) < 0)
			{
				last_status = errno;
				return;
			}
		}

		// allocate memory for buffer.
		char buff[2];
		// read bytes from input stream and write them to output stream.
		while(read(read_in, &buff, 1)) {
			buff[strlen(buff)] = '\0';
			// printf("%lu\n", strlen(buff));
			write(write_out, &buff, 1);
		}

		// Close input stream and check for errors.
		if(read_in != 0 && (close(read_in) < 0)) 
		{
			last_status = errno;
		}

		// close output stream and check for errors.
		if(write_out != 1 && (close(write_out) < 0)) 
		{
			last_status = errno;
		}
		fflush(stdout);
		return;
	}
	// If control reached this point, the command is external.

	int redir_in, redir_out;
	redir_in = redir_out = 0;

	int fd_in = 0;
	if(*num_tokens > 1 && tokens[1][0] == '<')
	{
		fd_in = open(tokens[1] + 1, O_RDONLY);
		strcpy(tokens[1], tokens[1] + 1);
		redir_in = 1;
	}

	int fd_out = 1;
	if(*num_tokens > 2 && tokens[2][0] == '>')
	{
		// Check if file exists. If it does not, create.
		if(access(tokens[2] + 1, F_OK) == -1)
		{
			creat(tokens[2] + 1, S_IRWXU | S_IWUSR | S_IRGRP | S_IROTH);
		}
		fd_out = open(tokens[2] + 1, O_WRONLY);
		//strcpy(tokens[2], tokens[2] + 1);
		free(tokens[2]);
		tokens[2] = NULL;
		redir_out = 1;
	}

	// Fork child
	pid_t pid = fork();
	if(pid == 0) {
		
		if(!daemon)
		{
			//redirect stderr
			int fd = open("/dev/null", O_WRONLY);
			dup2(fd, 2);
		} 

		// Execute command
		if(redir_in)
		{
			dup2(0, fd_in);
		}
		if(redir_out)
		{
			dup2(fd_out, 1);
		}

		fflush(stdout);
		execvp(tokens[0], tokens);
		fflush(stdout);
		exit(0);
	}
	else
	{
		// If in parent, wait for child to finish.
		if(!daemon)
		{
			waitpid(pid, &last_status, 0);
			dup2(1, 1);
		}
		
	}
}


// tokenize: break parsed line into tokens
char **tokenize(char *line, int *num_tokens)
{
	// Allocate memory for buffer and matrix of tokens.
	char *buff = (char*)malloc(256*sizeof(char));
	char **res = (char**)malloc(256*sizeof(char*));

	// Initialize indices.
	int buff_index = 0;
	int res_index = 0;

	// Skip leading whitespace.
	int begin = 0;
	while(isspace(line[begin]))
	{
		begin++;
	}

	// Begin parsing and tokenizing
	for(int i = begin; i < strlen(line) - 1; i++)
	{
		// If space, save next token to results matrix.
		if(isspace(line[i]))
		{
			buff[buff_index] = '\0';
			res[res_index] = malloc((strlen(buff) + 1)*sizeof(char*));
			strcpy(res[res_index], buff);
			res_index++;
			buff_index = 0;
			(*num_tokens)++;

			// Skip all whitespace.
			while(isspace(line[i])) {
				i++;
			}
			i--;
		}
		// If parsed ", parse everything up to closing " as a single token.
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
		// Else, add next character to token buffer.
		else
		{
			buff[buff_index] = line[i];
			buff_index++;
		}
	}

	// On reaching end of line, put remaining contents of buffer into results matrix.
	buff[buff_index] = '\0';
	res[res_index] = malloc((strlen(buff) + 1)*sizeof(char*));
	strcpy(res[res_index], buff);
	res_index++;
	(*num_tokens)++;

	// Return results matrix.
	return res;
}

/*void sigchld_handler()
{
	pid_t pid;
	int status;

	while(1) {
		 pid = waitpid(-1, &status, WNOHANG);
	}
}*/

/*void sigchld_handler(int sig)
{
    pid_t p;
    int status;
    while ((p = waitpid(-1, &status, WNOHANG)) != -1)
    {

    }
}*/

void sigchld_handler(int signum)
{
	int pid, status, serrno;
	serrno = errno;
	while (1)
	{
		pid = waitpid(WAIT_ANY, &status, WNOHANG);
		if (pid < 0)
		{
  			break;
		}
		if (pid == 0)
		{
			break;
		}
	}
	errno = serrno;
}
