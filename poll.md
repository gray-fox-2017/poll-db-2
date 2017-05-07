<!-- Release 1  -->
<!-- 1. Hitung jumlah vote untuk Sen. Olympia Snowe yang memiliki id 524. : 22 -->
SELECT COUNT(id) as jumlah
FROM votes
WHERE politician_id = 524
<!-- 2. Sekarang lakukan JOIN tanpa menggunakan id `524`. Query kedua tabel votes dan congress_members. 22-->
SELECT *
FROM
	votes AS v
	JOIN congress_members as cm ON v.politician_id = cm.id
WHERE
	cm.name = 'Sen. Olympia Snowe'
<!-- 3. Sekarang gimana dengan representative Erik Paulsen? Berapa banyak vote yang dia dapatkan? 21-->
SELECT count(v.id)
FROM
	votes AS v
	JOIN congress_members as cm ON v.politician_id = cm.id
WHERE
	cm.name = 'Rep. Erik Paulsen'
<!-- 4. Buatlah daftar peserta Congress yang mendapatkan vote terbanyak. Jangan sertakan field `created_at` dan `updated_at`. top 3 terbanyak-->
SELECT cm.name, cm.party, cm.location, cm.grade_1996, cm.grade_current,cm.years_in_congress, cm.dw1_score,  (count(v.id)) as number_of_votes
FROM
	votes AS v
	JOIN congress_members as cm ON v.politician_id = cm.id
GROUP BY cm.name, cm.party, cm.location, cm.grade_1996, cm.grade_current,cm.years_in_congress, cm.dw1_score
ORDER BY number_of_votes DESC, cm.id ASC
LIMIT 3
<!-- 5. Sekarang buatlah sebuah daftar semua anggota Congress yang setidaknya mendapatkan beberapa vote dalam urutan dari yang paling sedikit. Dan juga jangan sertakan field-field yang memiliki tipe date. -->
SELECT cm.name, cm.party, cm.location, cm.grade_1996, cm.grade_current,cm.years_in_congress, cm.dw1_score,  (count(v.id)) as number_of_votes
FROM
	votes AS v
	JOIN congress_members as cm ON v.politician_id = cm.id
GROUP BY cm.name, cm.party, cm.location, cm.grade_1996, cm.grade_current,cm.years_in_congress, cm.dw1_score
ORDER BY number_of_votes ASC, cm.id ASC
LIMIT 3
<!-- Release 2  -->

<!-- 1. Siapa anggota Congress yang mendapatkan vote terbanyak? List nama mereka dan jumlah vote-nya. Siapa saja yang memilih politisi tersebut? List nama mereka, dan jenis kelamin mereka. 10.000 rows, Lydia Fritsch 16 voter_id-->
SELECT v.first_name,v.last_name,vs.politician_id, x.total_votes
FROM
	voters AS v
	JOIN votes AS vs on v.id = vs.voter_id
	JOIN (
		SELECT count(politician_id) as total_votes, politician_id
		FROM votes
		GROUP BY politician_id
		ORDER BY total_votes DESC
	) x on vs.politician_id = x.politician_id
	ORDER BY x.politician_id  ASC
<!-- 2. Berapa banyak vote yang diterima anggota Congress yang memiliki grade di bawah 9 (gunakan field `grade_current`)? Ambil nama, lokasi, grade_current dan jumlah vote. 35 rows-->
SELECT  v.politician_id, cm.name, cm.location,cm.grade_current, count(v.politician_id) as total_votes
FROM
	votes AS v
	JOIN congress_members AS cm ON v.politician_id = cm.id
WHERE
	cm.grade_current < 9
GROUP BY v.politician_id, cm.name, cm.location,cm.grade_current
ORDER BY cm.grade_current DESC

<!-- 3. Apa saja 10 negara bagian yang memiliki voters terbanyak? List semua orang yang melakukan vote di negara bagian yang paling populer. (Akan menjadi daftar yang panjang, kamu bisa gunakan hasil dari query pertama untuk menyederhanakan query berikut ini.) -->
SELECT  vrs.first_name || ' ' || vrs.last_name AS voters_name,vrs.gender,v.politician_id, v.politician_name, v.location, v.total_votes
FROM
  (
    SELECT count(politician_id) AS total_votes, politician_id, location, dcm.name AS politician_name
    FROM votes dv
    JOIN congress_members dcm ON dv.politician_id = dcm.id
    WHERE location in (
      select location FROM (
        -- top 10 negara bagian terbanyak voters
        SELECT count(location) AS totLoc, location
        FROM congress_members
        GROUP BY location
	      ORDER BY totLoc DESC
        LIMIT 10
      )
    )
    GROUP BY politician_id
    ORDER BY total_votes DESC
  ) AS v
  JOIN votes AS v2 ON v.politician_id = v2.politician_id
	JOIN voters AS vrs ON vrs.id = v2.voter_id
ORDER BY total_votes DESC
<!-- 4. List orang-orang yang vote lebih dari dua kali. Harusnya mereka hanya bisa vote untuk posisi Senator dan satu lagi untuk wakil. Wow, kita dapat si tukang curang! Segera laporkan ke KPK!! -->
SELECT vrs.first_name, vrs.last_name, vrs.gender, curang.total_votes, cm.name
FROM
  voters AS vrs
  JOIN (
    SELECT count(politician_id) as total_votes, voter_id, politician_id
    FROM
      votes
    GROUP BY voter_id, politician_id
    HAVING total_votes >1
  ) AS curang on curang.voter_id = vrs.id
  JOIN congress_members AS cm ON cm.id = curang.politician_id
  GROUP BY vrs.first_name, vrs.last_name, vrs.gender, cm.name
  ORDER BY cm.name
<!-- 5. Apakah ada orang yang melakukan vote kepada politisi yang sama dua kali? Siapa namanya dan siapa nama politisinya? -->
SELECT vrs.first_name, vrs.last_name, vrs.gender, curang.total_votes, cm.name
FROM
  voters AS vrs
  JOIN (
    SELECT count(politician_id) as total_votes, voter_id, politician_id
    FROM
      votes
    GROUP BY voter_id, politician_id
    HAVING total_votes = 2
  ) AS curang on curang.voter_id = vrs.id
  JOIN congress_members AS cm ON cm.id = curang.politician_id
  ORDER BY cm.name
