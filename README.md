# sql-murder-mystery
Resolvendo o desafio SQL Murder Mystery usando apenas queries SQL
**Caso Nº 0115 · SQL City P.D. · 15/01/2018**

> Investigação completa de um assassinato fictício, resolvida inteiramente através de queries SQL em um banco de dados relacional. Este documento registra cada passo: a query executada, o resultado obtido e o que ele significou para o caso.
> 

---

## 01 · A cena do crime

**Objetivo:** recuperar o relatório da cena do crime.

```sql
SELECT * FROM crime_scene_report
WHERE date = 20180115
  AND type = 'murder'
  AND city = 'SQL City';
```

**Resultado:** as imagens de segurança mostram 2 testemunhas — a primeira mora na última casa da *Northwestern Dr*; a segunda, Annabel, mora na *Franklin Ave*.

**Significado:** o relatório não aponta o assassino, mas dá o ponto de partida — duas testemunhas oculares a serem localizadas.

---

## 02 · Localizando a testemunha 1

```sql
SELECT * FROM person
WHERE address_street_name = 'Northwestern Dr'
ORDER BY address_number DESC
LIMIT 1;
```

| id | name | address | ssn |
| --- | --- | --- | --- |
| 14887 | Morty Schapiro | 4919 Northwestern Dr | 111564949 |

**Significado:** "última casa" = maior número da rua. `ORDER BY ... DESC LIMIT 1` resolve o enigma do endereço.

---

## 03 · Localizando a testemunha 2

```sql
SELECT * FROM person
WHERE address_street_name = 'Franklin Ave'
  AND name LIKE 'Annabel%';
```

| id | name | address | ssn |
| --- | --- | --- | --- |
| 16371 | Annabel Miller | 103 Franklin Ave | 318771143 |

**Significado:** `LIKE 'Annabel%'` filtra pelo nome parcial sem precisar saber o sobrenome antecipadamente.

---

## 04 · Depoimento — Morty Schapiro

```sql
SELECT * FROM interview
WHERE person_id = 14887;
```

> "Ouvi um disparo e vi um homem correndo com uma bolsa da Get Fit Now Gym. A matrícula começava com **'48Z'** — só membros **Gold** têm essas bolsas. Ele entrou num carro com placa contendo **'H42W'**."
> 

**Significado:** três pistas de uma vez — matrícula parcial, categoria obrigatória, e fragmento de placa.

---

## 05 · Depoimento — Annabel Miller

```sql
SELECT * FROM interview
WHERE person_id = 16371;
```

> "Vi o assassinato acontecer. Reconheci o assassino da minha academia — o vi treinando no dia **9 de janeiro**."
> 

**Significado:** ganho de uma data-âncora (09/01/2018) para cruzar com os check-ins da academia.

---

## 06 · Cruzando a pista da bolsa (48Z + Gold)

```sql
SELECT * FROM get_fit_now_member
WHERE id LIKE '48Z%'
  AND membership_status = 'gold';
```

| id | person_id | name | status |
| --- | --- | --- | --- |
| 48Z7A | 28819 | Joe Germuska | gold |
| 48Z55 | 67318 | Jeremy Bowers | gold |

**Significado:** de toda a cidade, restaram exatamente 2 suspeitos.

---

## 07 · O "empate técnico" do check-in

```sql
SELECT * FROM get_fit_now_check_in
WHERE membership_id LIKE '48Z%'
  AND check_in_date = 20180109;
```

| membership_id | date | check_in | check_out |
| --- | --- | --- | --- |
| 48Z7A | 20180109 | 16:00 | 17:30 |
| 48Z55 | 20180109 | 15:30 | 17:00 |

**Significado:** os dois suspeitos bateram ponto no mesmo turno — a data sozinha não decide. Próximo passo: a placa do carro.

---

## 08 · A placa que engana (pista falsa)

```sql
SELECT * FROM drivers_license
WHERE plate_number LIKE 'H42W%';
```

| id | gender | hair | plate | car |
| --- | --- | --- | --- | --- |
| 183779 | female | blonde | H42W0X | Toyota Prius |

```sql
SELECT * FROM person
WHERE license_id = 183779;
```

| id | name | address |
| --- | --- | --- |
| 78193 | Maxine Whitely | 110 Fisk Rd |

**Significado:** Maxine Whitely não tinha depoimento algum na tabela `interview` — beco quase sem saída. Mas seu endereço (Fisk Rd) reaparece na próxima etapa.

---

## 09 · A vizinhança decide

- **Joe Germuska** → mora em *Fisk Rd 111*, vizinho de Maxine Whitely. **Sem depoimento registrado.**
- **Jeremy Bowers** → sem relação com Fisk Rd ou com a proprietária do carro. **Possui depoimento — uma confissão.**

**Significado:** coincidência de endereço não é prova. A tabela `interview` é quem decide: só Jeremy Bowers falou algo.

---

## 10 · A confissão de Jeremy Bowers

```sql
SELECT * FROM interview
WHERE person_id = 67318;
```

> "Fui contratado por uma mulher rica. Não sei o nome, mas ela tem entre **1,65m e 1,70m**, **cabelo ruivo**, dirige um **Tesla Model S**. Foi ao **SQL Symphony Concert** 3 vezes em **dezembro de 2017**."
> 

**Primeira resposta submetida:**

```sql
INSERT INTO solution VALUES (1, 'Jeremy Bowers');
```

**Resultado do jogo:** confirma Jeremy como o executor, mas revela que existe uma mandante por trás — o caso continua.

---

## 11 · Perfilando a mandante

```sql
SELECT * FROM drivers_license
WHERE gender = 'female'
  AND hair_color = 'red'
  AND car_make = 'Tesla' AND car_model = 'Model S'
  AND height BETWEEN 65 AND 67;
```

| id | age | height | hair | car |
| --- | --- | --- | --- | --- |
| 202298 | 68 | 66 | red | Tesla Model S |
| 291182 | 65 | 66 | red | Tesla Model S |
| 918773 | 48 | 65 | red | Tesla Model S |

**Significado:** o perfil físico + o carro reduziram o universo de suspeitas a apenas 3 pessoas.

---

## 12 · Colocando nomes nos rostos

```sql
SELECT * FROM person
WHERE license_id IN (202298, 291182, 918773);
```

| id | name | address |
| --- | --- | --- |
| 78881 | Red Korb | 107 Camerata Dr |
| 90700 | Regina George | 332 Maple Ave |
| 99716 | Miranda Priestly | 1883 Golden Ave |

**Significado:** três nomes, três álibis possíveis. Falta o critério do concerto para desempatar.

---

## 13 · O veredito final

```sql
SELECT * FROM facebook_event_checkin
WHERE person_id IN (78881, 90700, 99716)
  AND event_name = 'SQL Symphony Concert'
  AND date BETWEEN 20171201 AND 20171231;
```

| person_id | event | date |
| --- | --- | --- |
| 99716 | SQL Symphony Concert | 20171206 |
| 99716 | SQL Symphony Concert | 20171212 |
| 99716 | SQL Symphony Concert | 20171229 |

**Resposta final:**

```sql
INSERT INTO solution VALUES (1, 'Miranda Priestly');
```

**Resultado:** *"Congrats, you found the brains behind the murder! Everyone in SQL City hails you as the greatest SQL detective of all time."*

**A mandante era Miranda Priestly.**

---

## 14 · O que fica do caso

- Nem toda pista que "bate" é a resposta — o check-in do dia 9 quase enganou.
- Coincidência de endereço não é prova; a tabela certa (`interview`) é quem decide.
- Cada `WHERE` é uma hipótese sendo testada, não só um filtro de dados.
- A resposta óbvia (Jeremy Bowers) era só metade da história.

---

## 📎 Material de apoio

- Este documento serve como a versão completa e detalhada para quem quiser se aprofundar em cada query.
