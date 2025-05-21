#define _CRT_SECURE_NO_WARNINGS // 보안 경고 무시
#include <stdio.h> // 표준 입출력
#include <stdlib.h> // exit 함수 사용
#include <string.h> // 문자열 처리
#include <windows.h> // 콘솔 색상 사용
#include <conio.h> // 키보드 입력

#define ROWS 100 // 최대 맵 크기
#define COLS 100 // 최대 맵 크기

char map[ROWS][COLS]; // 맵 배열
int map_rows = 0, map_cols = 0; // 맵 크기
int px = 1, py = 1; // 플레이어 위치
int hp = 5; // 플레이어 체력
int current_map = 0; // 현재 맵 번호


const char* map_files[] = { // 맵 파일 경로 리스트 자신의 경로에 따라 바꾸시길 바랍니다.
	"C:\\Users\\suyou\\OneDrive\\바탕 화면\\대학 수업 메모\\C프로그래밍 자료\\map1.txt",
	"C:\\Users\\suyou\\OneDrive\\바탕 화면\\대학 수업 메모\\C프로그래밍 자료\\map2.txt",
	"C:\\Users\\suyou\\OneDrive\\바탕 화면\\대학 수업 메모\\C프로그래밍 자료\\map3.txt"
};
int total_maps = sizeof(map_files) / sizeof(map_files[0]); // 총 맵 개수

void setColor(int color) { // 콘솔 색상 설정
    SetConsoleTextAttribute(GetStdHandle(STD_OUTPUT_HANDLE), color); 
}

void load_map(const char* filename) { // 파일 들고오기
    FILE* fp = fopen(filename, "r");
    if (!fp) {
        perror("파일 열기 실패");
        exit(1);
    }

	map_rows = map_cols = 0;  // 맵 크기 초기화
	char line[COLS]; 
	while (fgets(line, sizeof(line), fp)) { // 한 줄씩 읽기
		size_t len = strlen(line); // 줄 길이 계산

		while (len > 0 && (line[len - 1] == '\n' || line[len - 1] == '\r')) { // 개행 문자 제거
            line[--len] = '\0'; 
        }

		strcpy_s(map[map_rows], sizeof(map[map_rows]), line); // 맵에 줄 저장
		map_rows++; // 맵 행 증가
		if (len > map_cols) map_cols = len; // 최대 열 길이 저장
    }

	fclose(fp); // 파일 닫기
}

void print_map() { // 맵 출력
	system("cls"); // 콘솔 화면 지우기
    for (int i = 0; i < map_rows; i++) {
        for (int j = 0; j < map_cols; j++) {
            if (i == px && j == py) {
                setColor(14); // 플레이어: 노란색
                printf("□");
            }
            else if (map[i][j] == '#') {
                setColor(1);  // 벽: 파란색
                printf("■");
            }
            else if (map[i][j] == '^') {
                setColor(12); // 함정: 빨간색
                printf("▲");
            }
            else if (map[i][j] == 'G') {
                setColor(10); // 목적지: 초록색
                printf("★");
            }
            else if (map[i][j] == '0') {
                setColor(11); // 입구: 하늘색
                printf("☆");
            }
            else if (map[i][j] == 'H') {
				setColor(13); // 생명력수급: 보라색
                printf("♥");
            }

            else {
				setColor(7);  // 기본 색상:하얀색
                printf("%c", map[i][j]);
            }
        }
        printf("\n");
    }
    setColor(7);
    printf("HP: %d\n", hp);
}

int retry_stage() {
    char retry;
    printf("해당 스테이지를 다시 시작하시겠습니까? (y/n): ");

	// 입력 받기
    if (scanf(" %c", &retry) != 1) {
        printf("입력 오류 발생\n");
        exit(1);
    }

    // 입력 버퍼 비우기
    int ch;
    while ((ch = getchar()) != '\n' && ch != EOF);

    return (retry == 'y' || retry == 'Y');
}
// 방향 이동 처리 함수
int move_player() {
    while (1) {
        char key = _getch();
        int new_px = px;
        int new_py = py;
        
		// 방향키 입력 처리
        if (key == 'w' || key == 'W') new_px--;
        else if (key == 's' || key == 'S') new_px++;
        else if (key == 'a'|| key == 'A') new_py--;
        else if (key == 'd' || key == 'D') new_py++;
        

        // 범위 밖 검사
        if (new_px < 0 || new_py < 0 || new_px >= map_rows || new_py >= map_cols)
            continue;

        // 벽(#)이면 이동 불가
        if (map[new_px][new_py] == '#')
            continue;

        // 함정(^) 밟으면 체력 감소
        if (map[new_px][new_py] == '^') {
            hp--;
            if (hp == 0) {
                print_map();
                printf("게임 오버\n");
				Sleep(1500); // 1.5초 대기
                return 0;
                
			}
			
        }
		// 하트(H) 밟으면 체력 회복
        if (map[new_px][new_py] == 'H') {
            hp++;
			map[new_px][new_py] = ' '; // 하트 제거
        }
        // ★ (G) 도착하면 게임 클리어
        if (map[new_px][new_py] == 'G') {
            px = new_px;
            py = new_py;
            print_map();
            printf("게임 클리어\n");
			Sleep(1500); // 1.5초 대기
            return 1; //클리어 처리
        }

        // 이동
        px = new_px;
        py = new_py;

        print_map();
    }
}

int main() {

    printf("세상에서 가장 쉬운 게임\n");
    Sleep(2000);
    printf("게임 방법: w, a, s, d로 이동(대문자도 가능)\n");
    Sleep(2000);
    printf("목표:★ 에 도착하기\n");
    Sleep(2000);
    printf("HP가 0이 되면 게임 오버입니다.\n");
    Sleep(2000);
    printf("함정: ▲ 밟으면 HP가 1 감소합니다.\n");
    Sleep(1000);
    printf("벽: ■\n");
    Sleep(1000);
    printf("하트: ♥ 밟으면 HP가 1 증가합니다.\n");
    Sleep(1000);
    printf("w: 위, s: 아래, a: 왼쪽, d: 오른쪽\n");
    Sleep(2000);

    printf("게임 시작\n");
    Sleep(1000);


    while (current_map < total_maps) {
		hp = 5; // 체력 초기화
		load_map(map_files[current_map]); // 맵 로드
		px = py = 1; // 플레이어 초기 위치
		print_map(); // 맵 출력

		int result = move_player(); // 플레이어 이동 처리

        if (result == 0) { // 게임 오버 시
            if (retry_stage()) {
                continue; // 맵 다시 시작
            }
            else {
                printf("게임 종료\n");
                exit(0);
            }
        }
		current_map++; // 다음 맵으로 이동
    }

	if (current_map == total_maps) { // 모든 맵 클리어
		printf("모든 맵을 클리어했습니다\n");
		Sleep(2000);
		exit(0); // 게임 종료
	}
	

    return 0;
}
