# 15_springJdbc



## Mail기능 추가하기 


### pom.xml 

    
``` 
  <dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-context-support</artifactId>
    <version>5.3.31</version>
  </dependency>

  <dependency>
    <groupId>com.sun.mail</groupId>
    <artifactId>javax.mail</artifactId>
    <version>1.6.2</version>
  </dependency>
```   

### EmailDTO,  EmailUtil만들기 

``` 
  public class EmailDto {
  private String from;
  private String receiver;
  private String text;
  private String subject;
} + getter setter 
```  

``` 
  public class EmailUtil {

	  @Autowired
    private JavaMailSender mailSender;

    public String sendMail(EmailDto email) {
        try {
            MimeMessage message = mailSender.createMimeMessage();
            MimeMessageHelper messageHelper = new MimeMessageHelper(message, true, "UTF-8");
            messageHelper.setTo(email.getReceiver());
            messageHelper.setText(email.getText());
            messageHelper.setFrom(email.getFrom());
            messageHelper.setSubject(email.getSubject());	// 메일제목은 생략이 가능하다

            mailSender.send(message);

        } catch(Exception e){
            System.out.println(e);
            return "Error";
        }
        return "Sucess";
    }
}

```  

### 사용하기
(이번 프로젝트의 경우에는 비밀번호 찾기기능을 구현하는 servce에서 사용한다! )

```  
  	public boolean findPasswd(HashMap<String,String> params) {
		
		System.out.println("memberId : : " + params.get("memberId"));
		System.out.println("member : : " + params.get("email"));
		int result = memberDao.findMember(params.get("memberId"),params.get("email"));
		
		System.out.println("result ="+result);
		if(result ==1) {
			// 랜덤한 문자열 생
			String  uuid = UUID.randomUUID().toString();
			System.out.println("newPw1 :" + uuid);
			
			// 필요없는 문자 제거
			String newPw = uuid.replaceAll("-", "");
			System.out.println("newPw 2: " +newPw);
			
			//암호화 
			String encodePw = Sha512Encoder.getInstance().getSecurePassword(newPw);
			System.out.println("newPw 3:" + encodePw);
			
			int updateResult = memberDao.updatePasswd(newPw,params.get("memberId"), params.get("email"));
			
			EmailDto emailDto = new EmailDto();
			emailDto.setFrom("whdudgms321@naver.com");
			emailDto.setReceiver("whdudgms123@naver.com");
			emailDto.setSubject("임시 비밀번호를 전송해드립니다.");
			// 이메일로 먼저 비번 알려주고 바꿔야 데이터 수정은 가장 마지막에 해야한다 
			emailDto.setText("dkdkdkdk");
			
			try {
				// 이메일 발송 실패 시 예외처리 
				emailUtil.sendMail(emailDto);
			}catch (Exception e){
				e.printStackTrace();
			}
			
			//to-do 임시 비밀번호로 업데이트
		//사용자 테이블에 비밀번호 컬럼 수정하는 메서드 작성.
			// interface > impl > service 
			// update lecture.member set passwd =  ?
			// where memberId = ? and email = ? 
			
			
			return updateResult ==1;
			
		}else {
			//ID/email 맞는 사용자는 무조건 1명이어야 함.
			return false;
		}    

	}
```  
