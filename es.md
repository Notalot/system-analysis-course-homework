# Описание системы Make Cats Free (MCF)

> [US-010] Все коты-тестировщики (клиенты) автоматически попадают в систему. 

Actor: <System>, event Test.TestPassed
Command: Create account
Data: Account
Event: Account.Created

Actor: Account (client)
Command: Login to system
Data: -
Event: Account.Logined

> [US-010] ... Для них доступен дашборд с заказанными услугами и кнопкой «запросить помощь».

Actor: Account (client)
Command: Create order
Data: Order
Event: Order.Created

> [US-021] Типы услуги задаются в админке менеджерами.

Actor: Account (manager)
Command: Create order type
Data: Type
Event: -

> [US-072] Матчинг принимает задачу и эталонный образец, а на выход выдаёт лучшего кандидата под задачу. Именно этого кандидата система автоматически назначает на задачу. И рассчитывает стоимость услуги.
Updated: матчинг всегда достает воркера, ситуаций, когда воркер не нашелся под задачу не существует.

Actor: event Order.Created
Command: Find worker
Data: Order + Account public id
Event: Match.WorkerFound

Actor: event Match.WorkerFound
Command: Calculate price
Data: Order + Account public id (client) + Account public id (worker)
Event: Match.PriceCalculated

> [US-090] Для прохождения тестирования необходимо оставить заявку на сайте, после чего с котом связывается менеджер и назначает набор тестов, который оценит уровень потенциального кандидата.

Actor: <Any>
Command: Test request
Data: ? telephone number
Event: Test.RequestCreated

> [US-110] Менеджеры должны иметь возможность конфигурировать набор тестов под каждого отдельного кота или вакансию котов.

Actor: Account (manager)
Command: Configure test set for a request
Data: Request public id + Tests set
Event: Test.TestsSetCreated

> [US-120] По результатам тестов кот либо добавляется в общий пул рабочих с полученными характеристиками (сильными/слабыми сторонами) и прочей информацией о коте (имя, возраст, порода, цвет шерсти, фото), либо бракуется.

Actor: <Any>
Command: Pass test
Data: Request public id + Test answers
Event: Test.TestPassed, Test.TestFailed

> [US-130] Если воркер не придёт на заказ по любой причине, заказ автоматически проваливается и клиенту предлагается сделать новый заказ (по сути копию) на другую удобную дату. При этом под заказ подбирается новый воркер.
Updated: после отмены, когда создался новый заказ, пользователь должен в ручную выбрать новую удобную дату, после чего под заказ автоматически подбирается воркер по инновационной системе подбора. После этого, заказ живет работает как любой другой.

Actor: <Task scheduler>
Command: Cancel order
Data: Account public id + Order public id
Event: Order.Canceled

> [US-150] Перед выездом на заказ воркер обязан получить необходимые для заказа расходники, включая бланк приёма работы. Для этого нам необходимо сказать отделу сборки расходников, кому и подо что нужны расходники.

Actor: Account (worker)
Command: Request equipment
Data: Order public id + Account public id
Event: Stock.RequestCreated

> [US-160] Сотрудники отдела расходников сами собирают заказы на свой вкус, нам хватит только уведомить сотрудников о новом заказе и предоставить кнопку «выдано» под каждый заказ.

Actor: Account (stock staff)
Command: Complete request
Data: Order public id + Account public id
Event: Stock.RequestCompleted

> [US-180] Когда воркер придёт в нужное время на заказ — ему необходимо отметиться в приложении (прислать фотографию и нажать кнопку «приступил»). В конце работы необходимо прислать фотографию бумажного отчёта с подписью клиента, что всё выполнено и претензий нет. 

Actor: Account (worker)
Command: Start order
Data: Order public id + Account public id + photo
Event: Order.Started

Actor: Account (worker)
Command: Finish order
Data: Order public id + Account public id + customer sign
Event: Order.Finished, Order.Failed

> [US-200] Все отменённые или провальные заказы автоматически попадают в отдел по изучению качества работы. Менеджеры пытаются понять, что случилось и как исправить ситуацию. Для этого могут связаться с клиентом.

Actor: event Order.Failed, event Order.Canceled
Command: Create audit record
Data: Order public id
Event: Audit.RecordCreated
  
> [US-210] Результат проверки качества заносится в отдельную форму, благодаря которой компания получает набор гипотез для улучшения бизнеса.

Actor: Account (manager)
Command: Save audit results
Data: Audit record + results
Event: Audit.ResultsSaved

> [US-220] Каждую неделю (в воскресенье) у клиента списывается сумма, равная общим затратам на запрошенные задачи. Для этого выставляется инвойс, и дальше происходит процесс списания через заданный клиентом способ.

Actor: <Task scheduler>
Command: Create withdraw invoice
Data: Account public id + sum
Event: Billing.WithdrawInvoiceCreated
  
Actor: event Billing.WithdrawInvoiceCreated
Command: Withdraw
Data: Withdraw method + sum
Event: Billing.Withdrew, Billing.WithdrawalFailed
  
> [US-230] Каждый последний день месяца воркеру начисляется сумма, равная общим выплатам минус все штрафы за текущий месяц. Для этого выставляется инвойс, и дальше происходит процесс зачисления средств через золотую шляпу.

Actor: <Task scheduler>
Command: Create invoice to pay
Data: Account public id + sum
Event: Billing.PayInvoiceCreated
  
Actor: event Billing.PayInvoiceCreated
Command: Pay
Data: Pay method + sum
Event: Billing.Paid, Billing.PayFailed
  
> [US-270] Иногда менеджер может зачислить любое количество денег для выбранного воркера на своё усмотрение. Надо знать, кто и за что начислил деньги.
  
Actor: Account (manager)
Command: Pay additionally
Data: Account public id + Account (worker) public id + sum + description
Event: Billing.AdditionalPayCreated
  
> [US-290] Вся логика ставок сводится к тому, что менеджеры ставят сумму на заказ и условие. В поле выполнения или отмены или провала заказа система автоматически рассчитывает, кому и сколько денег надо отдать (банк делится между победителями). Это не самый критически важный проект для компании, поэтому он вряд ли будет часто меняться.
  
Actor: Account (manager)
Command: Make a bet
Data: Account public id + Order public id + order result
Event: Betting.BetCreated
  
Actor: event Order.Finished, event Order.Failed, event Order.Canceled
Command: Calculate prize
Data: Account public ids + sum
Event: Betting.PrizeCalculated
