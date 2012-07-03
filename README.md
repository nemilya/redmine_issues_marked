redmine_issues_marked
=====================

Redmine плагин. Подсвечивающий задачи, которые пользователь ещё не просмотрел,
либо в которых есть какие-либо изменения.


Абстрактная постановка
======================

Есть:

* issues - задачи (поле changed_at)
* users - пользователи
* issues_users - таблица с полями issue_id, user_id, viewed_at - хранит
время когда пользователь просматривал задачу

Есть список досупных задач пользователя.

Сценарий:

Просмотр списка задач
---------------------

При просмотре списка задач, для строки задачи, если 

    issue_need_to_view?(cur_user, issue)

то подсвечивать эту строку.

Логическая реализация:


    def issue_need_to_view?(cur_user, issue)
      # последнее время просмотра задачи
      last_viewed_at = issues_users(cur_user, issue).viewed_at

      # если не просматривали совсем - то тем более
      return true if viewed_at.nil?

      issue.changed_at > last_viewed_at
    end

То есть если в задаче произошли изменения, а время последнего просмотра раньше.


Просмотр карточки задачи
------------------------

Выставить время viewed_at в таблицу issues_users

Логическая реалзиция:

    def mark_viewed(cur_user, issue)
      IssueUser.update_viewed_at_or_create(cur_user, issue, Time.now)
    end


Обновление задачи
-----------------

При изменении полей задачи, либо изменеии каких-либо вложенных объектов,
менять время модификации:


    def mark_issue_update(issue)
      issue.changed_at = Time.now
    end

    Issue.after_save = mark_issue_update self
    ....after_create = mark_issue_update ...


