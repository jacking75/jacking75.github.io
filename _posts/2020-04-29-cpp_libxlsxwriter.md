---
layout: post
title: C++ - libxlsxwriter를 사용하여 C++로 xlsx 파일 쓰기
published: true
categories: [C++]
tags: c++ libxlsxwriter excel
---
## libxlsxwriter
C용의 간단한 xlsx 쓰기 라이브러리.  
[Github 저장소](https://github.com/jmcnamara/libxlsxwriter.git )에서 공개하고 있다.  

풍부한 예제가 들어가 있다.  
  
  
## 설치
설치 하지 않아도 이용 가능.  
이 경우는 패스 설정이 필요.  

```
$ git clone https://github.com/jmcnamara/libxlsxwriter.git
$ cd libxlsxwriter
$ make
$ sudo make install
```
  
  
### 사용 방법
태그 구불의 표준 입력을 태그를 셀의 열 구분으로 하고, 인수에서 지정한 파일에 xlsx 형식으로 출력한다.  
  
```
tsv2xlsx.cpp
#include <fstream>
#include <iostream>
#include <string>
#include <vector>
#include <boost/algorithm/string.hpp>
#include "xlsxwriter.h"

int main(int argc, char **argv) {
  // 인수 조가
  if (argc < 2){
    std::cerr << "Usage: " << argv[0] << " output_filename.xlsx" << std::endl;
  }

  // Excel의 Workbook 과 Worksheet를 만든다
  lxw_workbook  *workbook  = workbook_new(argv[argc-1]); // argv[argc-1]은 파일 이름
  lxw_worksheet *worksheet = workbook_add_worksheet(workbook, NULL);

  std::vector<std::string> v_string;
  int row = 0;
  char input_text[51200]; // 표준 입력 버퍼(string 타입으로 표준 입력을 받는다)
  while (std::cin.getline(input_text, sizeof(input_text))){
    row++;
    // 행 수 상한을 넘지 않도록 한다.
    if(row == 1000000){
      break;
    }
    // 입력이 빈 행인 경우 쓰지 않는다
    if(!input_text[0]){
      continue;
    }
    // 입력을 태그로 구분한다.
    boost::split(v_string, input_text, boost::is_any_of("\t"));

    // 1행 출력 
    for (int col = 0; col < v_string.size(); col++){
      worksheet_write_string(worksheet, row, col, v_string[col].c_str(), NULL);
    }
  }

  workbook_close(workbook);

  return 0;
}
```
  
  
  
### 컴파일과 실행
  
```
% g++ tsv2xlsx.cpp -o tsv2xlsx -lxlsxwriter
% cat test.tsv | ./tsv2xlsx test.xlsx
```  
  