## Aula 36 - Listando horários disponíveis

Criar um controller para mostrar os horários disponíveis do prestador de serviço de uma dia
Quero saber todos os horários disponíveis do prestador para um determinado dia.

- Criar uma rota :
```
routes.get('/providers/:providerId/available', AvailableController.index);
```
Criar um controller `AvailableController.js`;

```
import {
  startOfDay,
  endOfDay,
  setHours,
  setMinutes,
  setSeconds,
  format,
  isAfter,
} from 'date-fns';
import { Op } from 'sequelize';
import Appointment from '../models/Appointment';

class AvailableController {
  async index(req, res) {
    const { date } = req.query;

    if (!date) {
      return res.status(400).json({ error: 'Invalid date' });
    }

    const searchDate = Number(date);

    // 2019-09-18 10:49:44

    const appointments = await Appointment.findAll({
      where: {
        provider_id: req.params.providerId,
        canceled_at: null,
        date: {
          [Op.between]: [startOfDay(searchDate), endOfDay(searchDate)],
        },
      },
    });

    const schedule = [
      '08:00', // 2019-09-18 08:00:00
      '09:00', // 2019-09-18 09:00:00
      '10:00', // 2019-09-18 10:00:00
      '11:00', // ...
      '12:00',
      '13:00',
      '14:00',
      '15:00',
      '16:00',
      '17:00',
      '18:00',
      '19:00',
    ];

    const available = schedule.map(time => {
      const [hour, minute] = time.split(':');
      const value = setSeconds(
        setMinutes(setHours(searchDate, hour), minute),
        0
      );

      return {
        time,
        // format to: 2019-09-18T15:40:44-04:00
        value: format(value, "yyyy-MM-dd'T'HH:mm:ssxxx"),
        available:
          isAfter(value, new Date()) &&
          !appointments.find(a => format(a.date, 'HH:mm') === time),
      };
    });

    return res.json(available);
  }
}

export default new AvailableController();
```

Só vai retornar os horários que não tem appointment marcado e que o valor desejado será posterior a data atual (isto é não se pode marcar um agendamento para um horário que já passou).

Recebo da requisição o ID do prestador e o dia que o usuário quer ver os horários disponíveis.

Essa data vem como timestamp e formato de string do componente datepicker do frontend. Então é preciso transformar um Number.

Depois busco todos agendamentos do provider informado pelo parâmetro da requisição, que não estejam cancelados, e que a data seja entre a primeira e última hora do dia informado.

Crio uma tabela estática de horários fixos.

E faço o restante da lógica e retorno para o usuário os horários em um objeto que retorna:

```
 {
    "time": "15:00",
    "value": "2019-09-18T15:00:00-04:00",
    "available": false
  },
  {
    "time": "16:00",
    "value": "2019-09-18T16:00:00-04:00",
    "available": true
  },
  {
    "time": "17:00",
    "value": "2019-09-18T17:00:00-04:00",
    "available": true
  },
```

Fim: [https://github.com/tgmarinho/gobarber/tree/aula36](https://github.com/tgmarinho/gobarber/tree/aula36)
