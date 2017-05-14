<!-- Release 1  -->

<!-- 1. Hitung jumlah vote untuk Sen. Olympia Snowe yang memiliki id 524. -->
SELECT COUNT(id) FROM votes WHERE politician_id = 524;

<!-- 2. Sekarang lakukan JOIN tanpa menggunakan id `524`. Query kedua tabel votes dan congress_members. -->
SELECT * FROM congress_members JOIN votes ON congress_members.id = votes.politician_id WHERE congress_members.name = 'Sen. Olympia Snowe';

<!-- 3. Sekarang gimana dengan representative Erik Paulsen? Berapa banyak vote yang dia dapatkan? -->
SELECT COUNT (votes.id) FROM votes JOIN congress_members ON congress_members.id = votes.politician_id WHERE congress_members.name LIKE '%Erik Paulsen%';

<!-- 4. Buatlah daftar peserta Congress yang mendapatkan vote terbanyak. Jangan sertakan field `created_at` dan `updated_at`. -->
SELECT congress_members.id, congress_members.name, congress_members.party,congress_members.location, congress_members.grade_1996, congress_members.grade_current, congress_members.years_in_congress, congress_members.dw1_score, COUNT(votes.id) FROM congress_members JOIN votes ON votes.politician_id = congress_members.id GROUP BY votes.politician_id ORDER BY COUNT(votes.id) DESC LIMIT 3;

<!-- 5. Sekarang buatlah sebuah daftar semua anggota Congress yang setidaknya mendapatkan beberapa vote dalam urutan dari yang paling sedikit. Dan juga jangan sertakan field-field yang memiliki tipe date. -->
SELECT congress_members.id, congress_members.name, congress_members.party,congress_members.location, congress_members.grade_1996, congress_members.grade_current, congress_members.years_in_congress, congress_members.dw1_score, COUNT(votes.id) FROM congress_members JOIN votes ON votes.politician_id = congress_members.id GROUP BY votes.politician_id ORDER BY COUNT(votes.id) ASC LIMIT 3;

<!-- Release 2  -->

<!-- 1. Siapa anggota Congress yang mendapatkan vote terbanyak? List nama mereka dan jumlah vote-nya. Siapa saja yang memilih politisi tersebut? List nama mereka, dan jenis kelamin mereka. -->
SELECT cm.name, COUNT(v.politician_id) FROM congress_members cm JOIN votes v ON cm.id = v.politician_id GROUP BY v.politician_id ORDER BY COUNT(v.politician_id) DESC;

SELECT congress_members.name, voters.first_name, voters.last_name, voters.gender, votes.politician_id FROM (votes JOIN voters ON voters.id = votes.voter_id) JOIN congress_members ON votes.politician_id = congress_members.id WHERE votes.politician_id IN (SELECT votes.politician_id FROM votes JOIN congress_members ON votes.politician_id = congress_members.id GROUP BY votes.politician_id ORDER BY COUNT(votes.politician_id) DESC LIMIT 3) ORDER BY votes.politician_id;

SELECT votes.politician_id, congress_members.name, sumtable.sum, voters.first_name, voters.last_name, voters.gender FROM votes JOIN voters ON votes.voter_id = voters.id JOIN (SELECT count(politician_id) as sum, politician_id FROM votes GROUP BY politician_id ORDER BY sum DESC) sumtable ON votes.politician_id = sumtable.politician_id JOIN congress_members ON congress_members.id = votes.politician_id ORDER BY sumtable.sum DESC;

<!-- 2. Berapa banyak vote yang diterima anggota Congress yang memiliki grade di bawah 9 (gunakan field `grade_current`)? Ambil nama, lokasi, grade_current dan jumlah vote. -->
SELECT congress_members.name, congress_members.location, congress_members.grade_current, COUNT(votes.politician_id) FROM congress_members JOIN votes ON congress_members.id = votes.politician_id WHERE congress_members.grade_current < 9 GROUP BY votes.politician_id ORDER BY COUNT(votes.politician_id) DESC;

<!-- 3. Apa saja 10 negara bagian yang memiliki voters terbanyak? List semua orang yang melakukan vote di negara bagian yang paling populer. (Akan menjadi daftar yang panjang, kamu bisa gunakan hasil dari query pertama untuk menyederhanakan query berikut ini.) -->
SELECT congress_members.location, COUNT(votes.id) FROM congress_members JOIN votes ON votes.politician_id = congress_members.id GROUP BY congress_members.location ORDER BY COUNT(votes.id) DESC LIMIT 10;

SELECT congress_members.location, voters.first_name, voters.last_name FROM congress_members JOIN votes ON votes.politician_id = congress_members.id JOIN voters ON voters.id = votes.voter_id WHERE congress_members.location IN (SELECT congress_members.location FROM congress_members JOIN votes ON votes.politician_id = congress_members.id GROUP BY congress_members.location ORDER BY COUNT(votes.id) DESC LIMIT 10) ORDER BY congress_members.location;

<!-- 4. List orang-orang yang vote lebih dari dua kali. Harusnya mereka hanya bisa vote untuk posisi Senator dan satu lagi untuk wakil. Wow, kita dapat si tukang curang! Segera laporkan ke KPK!! -->
SELECT voters.id, voters.first_name, voters.last_name, COUNT(votes.voter_id) FROM voters JOIN votes ON votes.voter_id = voters.id GROUP BY votes.voter_id HAVING COUNT(votes.voter_id) > 2 ORDER BY COUNT(votes.voter_id) DESC;

<!-- 5. Apakah ada orang yang melakukan vote kepada politisi yang sama dua kali? Siapa namanya dan siapa nama politisinya? -->
SELECT voters.id, voters.first_name, voters.last_name, COUNT(votes.politician_id), congress_members.name FROM voters JOIN votes ON votes.voter_id = voters.id JOIN congress_members ON congress_members.id = votes.politician_id GROUP BY votes.voter_id, votes.politician_id HAVING COUNT(votes.politician_id) > 1 ORDER BY COUNT(votes.politician_id) DESC;
