Vigenere cipher attack model
============================

코드 설명
---------
본 코드는 비즈네르 암호 방식을 바탕으로 modular addition 대신 XOR 연산을 이용하여 암호화된 ciphertext를 공격하는 모델이다.
### 암호화 방식
```
  unsigned char key [ KEY_LENGTH ] = { 0 x01 , 0 x02 , 0 x03 };
  .
  .
  .
  for ( int i = 0; fscanf(fpIn, "%c", &ch) != EOF; ++i) {
    ch ^= key[i % (sizeof(key)/sizeof(char))];
    fwrite(&ch , sizeof(ch), 1, fpOut);
  }

```
키의 길이와 각각의 아스키문자를 설정한후 plaintext 문자의 현재위치를 키의 길이로 나눈 나머지를 현재 문자와 XOR연산하여 암호화 한다.
### 공격 방법
```
for(int i = 1; i< MAX_KEY_LENGTH+1; i++){
  float arr[MAX_KEY_LENGTH][256]={0, };	
		
  for( int j=1; fscanf(fpIn, "%c", &ch) != EOF; j++){
  		arr[j%i][ch] +=1;
  }
  float a1= 0;
  float a2 = 0;
  float sum = 0;
  float prob = 0;
  for(int k = 0; k< i; k++){ 
  	sum=0;
  	for(int l = 0; l< 256; l++){
  	  sum += arr[k][l];
      a1 += arr[k][l]*(arr[k][l]);
    }	
    a2 += (sum*(sum));
  }

  prob = a1/a2;
  prob = fabs(prob-IC);
		
  if(prob <  max_prob){
	  max_prob=prob;
	  key_len=i;
  }
  fseek(fpIn, 0 ,SEEK_SET);
}
```
키의 길이를 index of coincidence를 구해 프리드먼 공격방식을 통해 예측한 키의 길이를 구한다.

```
float arr[15][256]={0,};
unsigned char key[15];
for(int j =0; fscanf(fpIn, "%c", &ch) != EOF; j++){
	arr[j%key_len][ch] +=1;
}
for(int j=0; j<key_len; j++){
	int max_index=0;
 	int max_alpha=0;
	for(int i = 0; i< 256; i++){
		if( max_index< arr[j][i]){
			max_index=arr[j][i];
			max_alpha=i;
    }
  }
  unsigned int white=32;
  unsigned int l= max_alpha;
	printf("l: %d\n", l);
	l ^= 32;
	key[j]=l;
	printf("j: %d k: %d\n", j, l);
}
```
예측한 키의 길이를 통해 i(키의 길이로 나눈 나머지)들의 문자들의 분포를 파악한다. 아스키코드를 통한 plaintext는 whitespace의 비율이 가장 높기 때문에 각각 가장 분포가 높은 문자를 whitespace 아스키고드(32)와 XOR연산하여 키를 구한다.

```
for(int i = 0; i< key_len; i++){
	fprintf(fpOut, "0x%x ", key[i]);
}
fprintf(fpOut, "\n");
fseek(fpIn, 0, SEEK_SET);

for(int i = 0; fscanf(fpIn, "%c", &ch) != EOF; ++i){
	ch ^= key[i%key_len];
	fwrite(&ch , sizeof(ch), 1 , fpOut );
}
```
예측한 키를 통해 주어진 ciphertext를 복호화한다.

## 설치 및 실행방법

얻은 ciphertext를 hw1_input.txt에 저장한다. 
attack.c는 hw1_input.txt를 읽기모드로 열고 프로그램을 실행하여 얻은 키와 복호화한 text를 hw1_output.txt에 쓴다.
