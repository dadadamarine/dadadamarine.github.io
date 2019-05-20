---
title: "[Linux/Ubuntu] File과 Directoriy 관련 명령어"
date: 2019-03-14 20:25:28 -0400
categories: Linux/Ubuntu
---

# File 

• A set of bytes to store data 

• Each file has a filename 

- –  A label referring to a particular file 
- –  Permitted characters include letters, digits, hyphens (-), underscores (_), and dots (.) 

> 요즘은 호환을 위해 다른문자도지원함

- –  Case-sensitive 

- –  File name starting with “.” means it is a hidden file 

  • The “ls” command lists the names of files 

### “ls” command

 – List the names of files 

• Options 

- –  -a : do not hide entries starting with “.” 
- –  -l : use a long listing format 
- –  -i : print index number of each file 
- –  -F : print file type (*: execution, /: directory, @: symbolic link) 
- –  -R : Recursively print directory contents 

-rw-rw-r-- 1 peterpan peterpan 57137735 Jun 10 2014 Floodlight.tar.gz

여기서 숫자 -> 파일일경우 하드 링크, 폴더일경우 안에있는 파일의 수





### Creating files with “cat” • “cat” command 

$ cat > shopping_list cucumber
 bread
 yoghurts 

- “>” sign means redirection of text types to the file “shopping_list” 

- Press “Ctrl+D” after a line break to denote the end of the file 

  – The next shell prompt is displayed 

- “ls” demonstrates the existence of the new file 

cat으로 파일 만들고, (cat > dde)

cat dde로 파일내용 볼 수 있음.

> '>' 의 의미 : redirect





