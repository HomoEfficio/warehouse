# Redmine 플러그인

- 설치 방법

    >- plugin 파일 다운로드 후 redmine/plugins/ 에 복사
    >- rake 명령 실행
    ```ruby
    rake redmine:plugins:migrate RAILS_ENV=production
    ```
    >- redmine 재시작

- 제거 방법

    >- rake 명령 실행
    ```ruby
    rake redmine:plugins:migrate NAME=<<<plugin_name>>> VERSION=0 RAILS_ENV=production
    ```

## 시각화

### Progressive Projects List

- URL :  http://stgeneral.github.io/redmine-progressive-projects-list/

- 프로젝트 별 진척 상태를 한눈에 확인 가능

![](http://stgeneral.github.io/redmine-progressive-projects-list/images/screenshots/v020/progressive-projects-list-v020-progress.png)


### Graphs

- URL :  http://www.redmine.org/projects/redmine/wiki/PluginGraphs

- 이슈 상태 변화를 한눈에 확인 가능

![](http://www.redmine.org/attachments/1736/open-aging-issues.png)


### Monitoring & Controlling

- URL : http://www.redmine.org/plugins/monitoring-controlling

- 이슈 현황 파이 차트 등 제공

![](http://ds6br8f5qp1u2.cloudfront.net/blog/wp-content/uploads/2015/08/monitoring-controlling-redmine-plugin.png?746e61)



##  이슈

### Checklist

- URL : http://www.redminecrm.com/projects/checklist/pages/1

- 이슈에 체크리스트 기능 추가

![](http://www.redminecrm.com/attachments/download/24403/view.png)


### Screenshot Paste

- URL : http://www.redmine.org/projects/redmine/wiki/PluginScreenshotPaste

- 클립보드에 있는 이미지를 레드마인 이슈 입력 폼에 붙여넣기 기능 추가


### CKEditor

- URL : https://www.redmine.org/plugins/ckeditor

- Wiki 편집을 WYSIWYG 방식으로 편리하게 해주는 기능 추가

![](http://blog.beany.co.kr/wp-content/uploads/2013/01/redmine_1.4.x_plugin_ckeditor_install_3.png)

- 주의
    - ckeditor 플러그인은 입력한 데이타를 Textile 로 변경해 주는게 아니라 HTML 을 바로 사용하므로 기존에 작성한 Textile 문서들은 제대로 표시가 안 될 수 있으며 반대로 ckeditor 플러그인 삭제시 작성한 문서를 Textile 로 재작성 해야 할 수 있다.
    - 프로젝트별로 ckeditor 플러그인 적용할 수 없으므로 전역적으로 사용하거나 사용하지 말아야 한다.



## Agile

### Agile plugin

- URL : http://www.redminecrm.com/projects/agile/pages/1

- Kanban이나 Scrum의 작업보드와 Burndown Chart, Velocity Chart, Lead time Chart 같은 기능 추가

![](http://www.redminecrm.com/attachments/download/11669/board.png)



## 테마

### RedmineCRM themes

- URL : http://www.redminecrm.com/pages/themes


### Flatly light Theme 

- URL : https://github.com/Nitrino/flatly_light_redmine

![](https://raw.githubusercontent.com/Nitrino/flatly_light_redmine/master/screenshots/screen_2.png)



## 기타

### Code Review

- URL : http://www.r-labs.org/projects/r-labs/wiki/Code_Review_en

- Repository Browser에 코드 리뷰 기능 추가

![](http://www.r-labs.org/attachments/download/84/edit.png)


### Timelog Timer

- URL : http://www.redmine.org/plugins/timelog_timer

- Timelog 폼에 타이머 기능 추가

![](http://www.redmine.org/attachments/download/8521/redmine_timelog_plugin.png)


### Q&A

- URL : http://www.redminecrm.com/projects/questions/pages/1

- 투표, 태그, Q&A 등의 기능이 추가된 포럼 플러그인

![](http://www.redminecrm.com/attachments/download/3751/forum.png)


### Usersnap

- URL : http://www.redmine.org/plugins/usersnap

- 사용자가 웹 애플리케이션 사용 상태를 캡쳐하고 버그 리포트를 쉽게 할 수 있게 해주는 기능 추가

![](http://ds6br8f5qp1u2.cloudfront.net/blog/wp-content/uploads/2015/08/usersnap-redmine-bug-tracking-plugin.jpg?746e61)


### 레드마인 한국 커뮤니티

http://www.redmine.or.kr/projects/5/boards/16












