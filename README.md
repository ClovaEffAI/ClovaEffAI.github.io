# Efficient AI팀 블로그
  - 참고 링크
    - https://github.com/mmistakes/minimal-mistakes (사용한 테마 링크)
    - https://velog.io/@eona1301/Github-Blog-Jekyll-minimal-mistakes (useful link, 한글)
    - https://github.com/mshlis/mshlis.github.io (예제)
  - posting하는 법
    - 기본적으로 markdown으로 작성한다.
    - _posts에 날짜-제목.md로 이루어진 파일을 생성한다. 1986-08-30-sejung-birthday.md
    - 해당 md 파일 안에 header를 다음처럼 넣는다.
      ```
      ---
      title: "Paper title / post title"
      author: "Se Jung Kwon" // _data/authors.yml 참고
      sidebar: true
      author_profile: true
      categories:
        - Paper Review  // Paper Review, Tech Review, ETC?
      tags: // 
        - Structured Pruning
        - Movement Pruning
        - BERT
        - Model Compression
        - ...
      ---
      ```
    - 논문 소개에 앞서서 '저자/학회 특이사항', 'Link'는 별도로 정리해주는 것이 좋을것 같다.
    - image를 넣고 싶으면 images 폴더에 식별자와 함께 넣고 링크를 건다.
      - 참고: https://rosejam.github.io/blogging/how-to-image/
    - 다음 post를 참고 할 것.
      - https://github.com/ClovaEffAI/ClovaEffAI.github.io/blob/main/_posts/2021-10-12-huggingface-bert-pruning.md
