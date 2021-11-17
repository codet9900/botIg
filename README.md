# botIg
impor unittest2

pembuka impor
impor openerp.osv.expression sebagai ekspresi
dari openerp.osv.expression impor get_unaccent_wrapper
dari openerp.osv.orm impor BaseModel
import openerp.tests.common sebagai umum

kelas test_expression(common.TransactionCase):

    def _reinit_mock(diri):
        self.query_list = daftar()

    def _mock_base_model_where_calc(self, model, *args, **kwargs):
        """ Buatan build_email tiruan agar dapat menguji nilainya. Simpan ke dalam
            beberapa variabel internal untuk pemrosesan terakhir. """
        self.query_list.append(self._base_model_where_calc(model, *args, **kwargs))
        # kembalikan kueri yang terakhir disimpan, yang ingin dijalankan oleh ORM
        kembalikan self.query_list[-1]

    pengaturan def (sendiri):
        super(test_expression, self).setUp()
        # Mock BaseModel._where_calc(), untuk dapat melanjutkan ke beberapa tes tentang ekspresi yang dihasilkan
        diri._reinit_mock()
        self._base_model_where_calc = BaseModel._where_calc
        BaseModel._where_calc = model lambda, cr, uid, argumen, konteks: self._mock_base_model_where_calc(model, cr, uid, args, konteks)

    def tearDown(diri):
        # Hapus ejekan
        BaseModel._where_calc = self._base_model_where_calc
        super(test_expression, self).tearDown()

    def test_00_in_not_in_m2m(mandiri):
        registry, cr, uid = self.registry, self.cr, self.uid

        # Buat 4 mitra tanpa kategori, atau satu atau dua kategori (dari dua kategori).

        kategori = registry('res.partner.category')
        cat_a = kategori.buat(cr, uid, {'name': 'test_expression_category_A'})
        cat_b = kategori.buat(cr, uid, {'name': 'test_expression_category_B'})

        mitra = registry('res.partner')
        a = partner.create(cr, uid, {'name': 'test_expression_partner_A', 'category_id': [(6, 0, [cat_a])]})
        b = partner.create(cr, uid, {'name': 'test_expression_partner_B', 'category_id': [(6, 0, [cat_b])]})
        ab = partner.create(cr, uid, {'name': 'test_expression_partner_AB', 'category_id': [(6, 0, [cat_a, cat_b])]})
        c = partner.create(cr, uid, {'name': 'test_expression_partner_C'})

        # Tes.

        # Pada bidang one2many atau many2many, `in` harus dibaca `contains` (dan
        # `tidak dalam` harus dibaca `tidak mengandung`.

        with_a = partner.search(cr, uid, [('category_id', 'in', [cat_a])])
        self.assertEqual(set([a, ab]), set(with_a), "Cari kategori_id di cat_a gagal.")

        with_b = partner.search(cr, uid, [('category_id', 'in', [cat_b])])
        self.assertEqual(set([ab, b]), set(with_b), "Cari kategori_id di cat_b gagal.")

        # Bermitra dengan kategori A atau kategori B.
        with_a_or_b = partner.search(cr, uid, [('category_id', 'in', [cat_a, cat_b])])
        self.assertEqual(set([ab, a, b]), set(with_a_or_b), "Penelusuran kategori_id berisi cat_a atau cat_b gagal.")

        # Tunjukkan bahwa `berisi daftar` benar-benar `berisi elemen atau berisi elemen`.
        with_a_or_with_b = partner.search(cr, uid, ['|', ('category_id', 'in', [cat_a]), ('category_id', 'in', [cat_b])])
        self.assertEqual(set([ab, a, b]), set(with_a_or_with_b), "Penelusuran kategori_id berisi cat_a atau berisi cat_b gagal.")

        # Jika kita mengubah OR di AND...
        with_a_and_b = partner.search(cr, uid, [('category_id', 'in', [cat_a]), ('category_id', 'in', [cat_b])])
        self.assertEqual(set([ab]), set(with_a_and_b), "Penelusuran kategori_id berisi cat_a dan cat_b gagal.")

        # Mitra tanpa kategori A dan tanpa kategori B.
        tanpa_a_or_b = partner.search(cr, uid, [('category_id', 'not in', [cat_a, cat_b])])
        self.assertTrue(all(i not in without_a_or_b for i in [a, b, ab]), "Penelusuran kategori_id tidak mengandung cat_a atau cat_b gagal (1).")
        self.assertTrue(c di without_a_or_b, "Penelusuran untuk kategori_id tidak mengandung cat_a atau cat_b gagal (2).")

        # Tunjukkan bahwa `tidak mengandung daftar` sebenarnya `tidak mengandung elemen dan tidak mengandung elemen`.
        tanpa_a_dan_tanpa_b = partner.search(cr, uid, [('category_id', 'not in', [cat_a]), ('category_id', 'not in', [cat_b])])
        self.assertTrue(all(saya tidak di without_a_and_without_b for i in [a, b, ab]), "Penelusuran untuk kategori_id tidak mengandung cat_a dan cat_b gagal (1).")
        self.assertTrue(c di without_a_and_without_b, "Cari kategori_id tidak mengandung cat_a dan cat_b gagal (2).")

        # Kami dapat mengecualikan mitra mana pun yang mengandung kategori A.
        tanpa_a = partner.search(cr, uid, [('category_id', 'not in', [cat_a])])
        self.assertTrue(a not in without_a, "Cari kategori_id tidak mengandung cat_a gagal (1).")
        self.assertTrue(ab not in without_a, "Cari kategori_id tidak mengandung cat_a gagal (2).")
        self.assertTrue(set([b, c]).issubset(set(without_a)), "Gagal mencari kategori_id tidak mengandung cat_a (3).")

        # (Jelas kita bisa melakukan hal yang sama untuk kategori B.)
        tanpa_b = partner.search(cr, uid, [('category_id', 'not in', [cat_b])])
        self.assertTrue(b tidak di without_b, "Pencarian untuk kategori_id tidak mengandung cat_b gagal (1).")
        self.assertTrue(ab not in without_b, "Cari kategori_id tidak mengandung cat_b gagal (2).")
        self.assertTrue(set([a, c]).issubset(set(without_b)), "Gagal mencari kategori_id tidak mengandung cat_b (3).")

        # Kami tidak dapat mengungkapkan hal berikut: Mitra dengan kategori yang berbeda dari A.
        # dengan_any_other_than_a = ...
        # self.assertTrue(a tidak di with_any_other_than_a, "Cari kategori_id dengan selain cat_a gagal (1).")
        # self.assertTrue(ab in with_any_other_than_a, "Cari kategori_id dengan selain cat_a gagal (2).")

    def test_10_expression_parse(sendiri):
        # Catatan TDE: tes tersebut telah ditambahkan saat memfaktorkan ulang metode ekspresi.parse().
        # Mereka datang sebagai tambahan dari test_osv_expression.yml yang sudah ada; mungkin beberapa tes
        # akan sedikit berlebihan
        registry, cr, uid = self.registry, self.cr, self.uid
        pengguna_obj = registry('res.users')

        # Buat pengguna
        a = users_obj.create(cr, uid, {'name': 'test_A', 'login': 'test_A'})
        b1 = users_obj.create(cr, uid, {'name': 'test_B', 'login': 'test_B'})
        b1_user = users_obj.browse(cr, uid, [b1])[0]
        b2 = users_obj.create(cr, uid, {'name': 'test_B2', 'login': 'test_B2', 'parent_id': b1_user.partner_id.id})

        # Test1: pewarisan sederhana
        user_ids = users_obj.search(cr, uid, [('name', 'like', 'test')])
        self.assertEqual(set(user_ids), set([a, b1, b2]), 'mencari melalui warisan gagal')
        user_ids = users_obj.search(cr, uid, [('name', '=', 'test_B')])
        self.assertEqual(set(user_ids), set([b1]), 'mencari melalui warisan gagal')

        # Test2: pewarisan + bidang relasional
        user_ids = users_obj.search(cr, uid, [('child_ids.name', 'like', 'test_B')])
        self.assertEqual(set(user_ids), set([b1]), 'mencari melalui warisan gagal')
        
        # Istimewa =? operator berarti "sama jika benar diatur, jika tidak selalu Benar"
        user_ids = users_obj.search(cr, uid, [('name', 'like', 'test'), ('parent_id', '=?', False)])
        self.assertEqual(set(user_ids), set([a, b1, b2]), '(x =? False) gagal')
        user_ids = users_obj.search(cr, uid, [('name', 'like', 'test'), ('parent_id', '=?', b1_user.partner_id.id)])
        self.assertEqual(set(user_ids), set([b2]), '(x =? id) gagal')

    def test_20_auto_join(mandiri):
        registry, cr, uid = self.registry, self.cr, self.uid
        tanpa aksen = get_unaccent_wrapper(cr)

        # Dapatkan model
        partner_obj = registry('res.partner')
        state_obj = registry('res.country.state')
        bank_obj = registry('res.partner.bank')

        # Dapatkan kolom uji
        partner_state_id_col = partner_obj._columns.get('state_id') # many2one di res.partner ke res.country.state
        partner_parent_id_col = partner_obj._columns.get('parent_id') # many2one di res.partner ke res.partner
        state_country_id_col = state_obj._columns.get('country_id') # many2one di res.country.state di res.country
        partner_child_ids_col = partner_obj._columns.get('child_ids') # one2many di res.partner ke res.partner
        partner_bank_ids_col = partner_obj._columns.get('bank_ids') # one2many di res.partner ke res.partner.bank
        kategori_id_kol = partner_obj._columns.get('category_id') # many2many di res.partner ke res.partner.category

        # Dapatkan jenis rekening bank pertama untuk dapat membuat res.partner.bank
        bank_type = bank_obj._bank_type_get(cr, uid)[0]
        # Dapatkan data negara/negara bagian
        country_us_id = registry('res.country').search(cr, uid, [('code', 'like', 'US')])[0]
        state_ids = registry('res.country.state').search(cr, uid, [('country_id', '=', country_us_id)], limit=2)

        # Buat data demo: mitra dan objek bank
        p_a = partner_obj.create(cr, uid, {'name': 'test__A', 'state_id': state_ids[0]})
        p_b = partner_obj.create(cr, uid, {'name': 'test__B', 'state_id': state_ids[1]})
        p_aa = partner_obj.create(cr, uid, {'name': 'test__AA', 'parent_id': p_a, 'state_id': state_ids[0]})
        p_ab = partner_obj.create(cr, uid, {'name': 'test__AB', 'parent_id': p_a, 'state_id': state_ids[1]})
        p_ba = partner_obj.create(cr, uid, {'name': 'test__BA', 'parent_id': p_b, 'state_id': state_ids[0]})
        b_aa = bank_obj.create(cr, uid, {'name': '__bank_test_a', 'state': bank_type[0], 'partner_id': p_aa, 'acc_number': '1234'})
        b_ab = bank_obj.create(cr, uid, {'name': '__bank_test_b', 'state': bank_type[0], 'partner_id': p_ab, 'acc_number': '5678'})
        b_ba = bank_obj.create(cr, uid, {'name': '__bank_test_b', 'state': bank_type[0], 'partner_id': p_ba, 'acc_number': '9876'})

        # -------------------------------------------------- -
        # Test1: dasar-dasar tentang atribut
        # -------------------------------------------------- -

        kategori_id_col._auto_join = Benar
        self.assertRaises(NotImplementedError, partner_obj.search, cr, uid, [('category_id.name', '=', 'foo')])
        kategori_id_col._auto_join = Salah

        # -------------------------------------------------- -
        # Tes2: one2many
        # -------------------------------------------------- -

        nama_tes = 'tes_a'

        # Lakukan: one2many tanpa _auto_join
        diri._reinit_mock()
        partner_ids = partner_obj.search(cr, uid, [('bank_ids.name', 'like', name_test)])
        # Hasil tes
        self.assertEqual(set(partner_ids), set([p_aa]),
            "_auto_join off: ('bank_ids.name', 'like', '..'): hasil salah")
        # Uji kueri yang dihasilkan
        self.assertEqual(len(self.query_list), 3,
            "_auto_join off: ('bank_ids.name', 'like', '..') harus menghasilkan 3 kueri (1 di res_partner_bank, 2 di res_partner)")
        sql_query = self.query_list[0].get_sql()
        self.assertIn('res_partner_bank', sql_query[0],
            "_auto_join off: ('bank_ids.name', 'like', '..') kueri pertama tabel utama salah")

        diharapkan = "%s::teks seperti %s" % (unaccent('"res_partner_bank"."name"'), unaccent('%s'))
        self.assertIn(diharapkan, sql_query[1],
            "_auto_join off: ('bank_ids.name', 'like', '..') query pertama salah dimana kondisi")
        
        self.assertEqual(set(['%' + name_test + '%']), set(sql_query[2]),
            "_auto_join off: ('bank_ids.name', 'like', '..') query pertama parameter salah")
        sql_query = self.query_list[2].get_sql()
        self.assertIn('res_partner', sql_query[0],
            "_auto_join off: ('bank_ids.name', 'like', '..') kueri ketiga tabel utama salah")
        self.assertIn('"res_partner"."id" di (%s)', sql_query[1],
            "_auto_join off: ('bank_ids.name', 'like', '..') permintaan ketiga salah di mana kondisinya")
        self.assertEqual(set([p_aa]), set(sql_query[2]),
            "_auto_join off: ('bank_ids.name', 'like', '..') parameter query ketiga salah")

        # Lakukan: mengalirkan one2many tanpa _auto_join
        diri._reinit_mock()
        partner_ids = partner_obj.search(cr, uid, [('child_ids.bank_ids.id', 'in', [b_aa, b_ba])])
        # Hasil tes
        self.assertEqual(set(partner_ids), set([p_a, p_b]),
            "_auto_join off: ('child_ids.bank_ids.id', 'in', [..]): hasil salah")
        # Uji kueri yang dihasilkan
        self.assertEqual(len(self.query_list), 5,
            "_auto_join off: ('child_ids.bank_ids.id', 'in', [..]) harus menghasilkan 5 kueri (1 di res_partner_bank, 4 di res_partner)")

        # Lakukan: one2many dengan _auto_join
        partner_bank_ids_col._auto_join = Benar
        diri._reinit_mock()
        partner_ids = partner_obj.search(cr, uid, [('bank_ids.name', 'like', 'test_a')])
        # Hasil tes
        self.assertEqual(set(partner_ids), set([p_aa]),
            "_auto_join on: ('bank_ids.name', 'like', '..') salah hasil")
        # Uji kueri yang dihasilkan
        self.assertEqual(len(self.query_list), 1,
            "_auto_join on: ('bank_ids.name', 'like', '..') harus menghasilkan 1 query")
        sql_query = self.query_list[0].get_sql()
        self.assertIn('"res_partner"', sql_query[0],
            "_auto_join on: ('bank_ids.name', 'like', '..') query tabel utama salah")
        self.assertIn('"res_partner_bank" sebagai "res_partner__bank_ids"', sql_query[0],
            "_auto_join on: ('bank_ids.name', 'like', '..') query join salah")

        diharapkan = "%s::teks seperti %s" % (unaccent('"res_partner__bank_ids"."name"'), unaccent('%s'))
        self.assertIn(diharapkan, sql_query[1],
            "_auto_join on: ('bank_ids.name', 'like', '..') query salah dimana kondisi")
        
        self.assertIn('"res_partner"."id"="res_partner__bank_ids"."partner_id"', sql_query[1],
            "_auto_join on: ('bank_ids.name', 'like', '..') query kondisi join salah")
        self.assertEqual(set(['%' + name_test + '%']), set(sql_query[2]),
            "_auto_join on: ('bank_ids.name', 'like', '..') query parameter salah")

        # Lakukan: one2many dengan _auto_join, tes akhir daun adalah id
        diri._reinit_mock()
        partner_ids = partner_obj.search(cr, uid, [('bank_ids.id', 'in', [b_aa, b_ab])])
        # Hasil tes
        self.assertEqual(set(partner_ids), set([p_aa, p_ab]),
            "_auto_join on: ('bank_ids.id', 'in', [..]) hasil salah")
        # Uji kueri yang dihasilkan
        self.assertEqual(len(self.query_list), 1,
            "_auto_join on: ('bank_ids.id', 'in', [..]) harus menghasilkan 1 query")
        sql_query = self.query_list[0].get_sql()
        self.assertIn('"res_partner"', sql_query[0],
            "_auto_join on: ('bank_ids.id', 'in', [..]) query tabel utama salah")
        self.assertIn('"res_partner__bank_ids"."id" di (%s,%s)', sql_query[1],
            "_auto_join on: ('bank_ids.id', 'in', [..]) query salah dimana kondisi")
        self.assertEqual(set([b_aa, b_ab]), set(sql_query[2]),
            "_auto_join on: ('bank_ids.id', 'in', [..]) query parameter salah")

        # Lakukan: 2 cascade one2many dengan _auto_join, tes akhir daun adalah id
        partner_child_ids_col._auto_join = Benar
        diri._reinit_mock()
        partner_ids = partner_obj.search(cr, uid, [('child_ids.bank_ids.id', 'in', [b_aa, b_ba])])
        # Hasil tes
        self.assertEqual(set(partner_ids), set([p_a, p_b]),
            "_auto_join on: ('child_ids.bank_ids.id', 'not in', [..]): hasil salah")
        # # Uji kueri yang dihasilkan
        self.assertEqual(len(self.query_list), 1,
            "_auto_join on: ('child_ids.bank_ids.id', 'in', [..]) harus menghasilkan 1 query")
        sql_query = self.query_list[0].get_sql()
        self.assertIn('"res_partner"', sql_query[0],
            "_auto_join on: ('child_ids.bank_ids.id', 'in', [..]) tabel utama salah")
        self.assertIn('"res_partner" sebagai "res_partner__child_ids"', sql_query[0],
            "_auto_join on: ('child_ids.bank_ids.id', 'in', [..]) query join salah")
        self.assertIn('"res_partner_bank" sebagai "res_partner__child_ids__bank_ids"', sql_query[0],
            "_auto_join on: ('child_ids.bank_ids.id', 'in', [..]) query join salah")
        self.assertIn('"res_partner__child_ids__bank_ids"."id" di (%s,%s)', sql_query[1],
            "_auto_join on: ('child_ids.bank_ids.id', 'in', [..]) query salah dimana kondisi")
        self.assertIn('"res_partner"."id"="res_partner__child_ids"."parent_id"', sql_query[1],
            "_auto_join on: ('child_ids.bank_ids.id', 'in', [..]) query kondisi join salah")
        self.assertIn('"res_partner__child_ids"."id"="res_partner__child_ids__bank_ids"."partner_id"', sql_query[1],
            "_auto_join on: ('child_ids.bank_ids.id', 'in', [..]) query kondisi join salah")
        self.assertEqual(set([b_aa, b_ba]), set(sql_query[2][-2:]),
            "_auto_join on: ('child_ids.bank_ids.id', 'in', [..]) query parameter salah")

        # -------------------------------------------------- -
        # Tes3: banyak2satu
        # -------------------------------------------------- -

        name_test = 'AS'

        # Lakukan: many2one tanpa _auto_join
        diri._reinit_mock()
        partner_ids = partner_obj.search(cr, uid, [('state_id.country_id.code', 'like', name_test)])
        # Hasil tes: setidaknya data tambahan + data demo kami
        self.assertTrue(set([p_a, p_b, p_aa, p_ab, p_ba]).issubset(set(partner_ids)),
            "_auto_join off: ('state_id.country_id.code', 'like', '..') hasil salah")
        # Uji kueri yang dihasilkan
        self.assertEqual(len(self.query_list), 3,
            "_auto_join off: ('state_id.country_id.code', 'like', '..') harus menghasilkan 3 kueri (1 di res_country, 1 di res_country_state, 1 di res_partner)")

        # Lakukan: many2one dengan 1 _auto_join di many2one pertama
        partner_state_id_col._auto_join = Benar
        diri._reinit_mock()
        partner_ids = partner_obj.search(cr, uid, [('state_id.country_id.code', 'like', name_test)])
        # Hasil tes: setidaknya data tambahan + data demo kami
        self.assertTrue(set([p_a, p_b, p_aa, p_ab, p_ba]).issubset(set(partner_ids)),
            "_auto_join on for state_id: ('state_id.country_id.code', 'like', '..') hasil salah")
        # Uji kueri yang dihasilkan
        self.assertEqual(len(self.query_list), 2,
            "_auto_join on for state_id: ('state_id.country_id.code', 'like', '..') harus menghasilkan 2 query")
        sql_query = self.query_list[0].get_sql()
        self.assertIn('"res_country"', sql_query[0],
            "_auto_join on for state_id: ('state_id.country_id.code', 'like', '..') query 1 tabel utama salah")

        diharapkan = "%s::teks seperti %s" % (unaccent('"res_country"."code"'), unaccent('%s'))
        self.assertIn(diharapkan, sql_query[1],
            "_auto_join on for state_id: ('state_id.country_id.code', 'like', '..') query 1 salah dimana kondisi")

        self.assertEqual(['%' + name_test + '%'], sql_query[2],
            "_auto_join on for state_id: ('state_id.country_id.code', 'like', '..') query 1 parameter salah")
        sql_query = self.query_list[1].get_sql()
        self.assertIn('"res_partner"', sql_query[0],
            "_auto_join on for state_id: ('state_id.country_id.code', 'like', '..') query 2 tabel utama salah")
        self.assertIn('"res_country_state" sebagai "res_partner__state_id"', sql_query[0],
            "_auto_join on untuk state_id: ('state_id.country_id.code', 'like', '..') query 2 salah join")
        self.assertIn('"res_partner__state_id"."country_id" di (%s)', sql_query[1],
            "_auto_join on for state_id: ('state_id.country_id.code', 'like', '..') query 2 salah dimana kondisi")
        self.assertIn('"res_partner"."state_id"="res_partner__state_id"."id"', sql_query[1],
            "_auto_join on for state_id: ('state_id.country_id.code', 'like', '..') query 2 kondisi join salah")

        # Lakukan: many2one dengan 1 _auto_join di many2one kedua
        partner_state_id_col._auto_join = Salah
        state_country_id_col._auto_join = Benar
        diri._reinit_mock()
        partner_ids = partner_obj.search(cr, uid, [('state_id.country_id.code', 'like', name_test)])
        # Hasil tes: setidaknya data tambahan + data demo kami
        self.assertTrue(set([p_a, p_b, p_aa, p_ab, p_ba]).issubset(set(partner_ids)),
            "_auto_join on for country_id: ('state_id.country_id.code', 'like', '..') hasil salah")
        # Uji kueri yang dihasilkan
        self.assertEqual(len(self.query_list), 2,
            "_auto_join on for country_id: ('state_id.country_id.code', 'like', '..') harus menghasilkan 2 query")
        # -- pertanyaan pertama
        sql_query = self.query_list[0].get_sql()
        self.assertIn('"res_country_state"', sql_query[0],
            "_auto_join on for country_id: ('state_id.country_id.code', 'like', '..') kueri 1 tabel utama salah")
        self.assertIn('"res_country" sebagai "res_country_state__country_id"', sql_query[0],
            "_auto_join on for country_id: ('state_id.country_id.code', 'like', '..') query 1 salah join")

        diharapkan = "%s::teks seperti %s" % (unaccent('"res_country_state__country_id"."code"'), unaccent('%s'))
        self.assertIn(diharapkan, sql_query[1],
            "_auto_join on for country_id: ('state_id.country_id.code', 'like', '..') query 1 salah dimana kondisi")
        
        self.assertIn('"res_country_state"."country_id"="res_country_state__country_id"."id"', sql_query[1],
            "_auto_join on for country_id: ('state_id.country_id.code', 'like', '..') query 1 kondisi join salah")
        self.assertEqual(['%' + name_test + '%'], sql_query[2],
            "_auto_join on for country_id: ('state_id.country_id.code', 'like', '..') query 1 parameter salah")
        # -- pertanyaan kedua
        sql_query = self.query_list[1].get_sql()
        self.assertIn('"res_partner"', sql_query[0],
            "_auto_join on for country_id: ('state_id.country_id.code', 'like', '..') query 2 tabel utama salah")
        self.assertIn('"res_partner"."state_id" di', sql_query[1],
            "_auto_join on for country_id: ('state_id.country_id.code', 'like', '..') query 2 salah dimana kondisi")

        # Lakukan: many2one dengan 2 _auto_join
        partner_state_id_col._auto_join = Benar
        state_country_id_col._auto_join = Benar
        diri._reinit_mock()
        partner_ids = partner_obj.search(cr, uid, [('state_id.country_id.code', 'like', name_test)])
        # Hasil tes: setidaknya data tambahan + data demo kami
        self.assertTrue(set([p_a, p_b, p_aa, p_ab, p_ba]).issubset(set(partner_ids)),
            "_auto_join on: ('state_id.country_id.code', 'like', '..') hasil salah")
        # Uji kueri yang dihasilkan
        self.assertEqual(len(self.query_list), 1,
            "_auto_join on: ('state_id.country_id.code', 'like', '..') harus menghasilkan 1 query")
        sql_query = self.query_list[0].get_sql()
        self.assertIn('"res_partner"', sql_query[0],
            "_auto_join on: ('state_id.country_id.code', 'like', '..') query tabel utama salah")
        self.assertIn('"res_country_state" sebagai "res_partner__state_id"', sql_query[0],
            "_auto_join on: ('state_id.country_id.code', 'like', '..') query join salah")
        self.assertIn('"res_country" sebagai "res_partner__state_id__country_id"', sql_query[0],
            "_auto_join on: ('state_id.country_id.code', 'like', '..') query join salah")

        diharapkan = "%s::teks seperti %s" % (unaccent('"res_partner__state_id__country_id"."code"'), unaccent('%s'))
        self.assertIn(diharapkan, sql_query[1],
            "_auto_join on: ('state_id.country_id.code', 'like', '..') query salah dimana kondisi")
        
        self.assertIn('"res_partner"."state_id"="res_partner__state_id"."id"', sql_query[1],
            "_auto_join on: ('state_id.country_id.code', 'like', '..') query kondisi join salah")
        self.assertIn('"res_partner__state_id"."country_id"="res_partner__state_id__country_id"."id"', sql_query[1],
            "_auto_join on: ('state_id.country_id.code', 'like', '..') query kondisi join salah")
        self.assertEqual(['%' + name_test + '%'], sql_query[2],
            "_auto_join on: ('state_id.country_id.code', 'like', '..') query parameter salah")

        # -------------------------------------------------- -
        # Test4: atribut domain pada bidang one2many
        # -------------------------------------------------- -

        partner_child_ids_col._auto_join = Benar
        partner_bank_ids_col._auto_join = Benar
        partner_child_ids_col._domain = lambda self: ['!', ('name', '=', self._name)]
        partner_bank_ids_col._domain = [('acc_number', 'like', '1')]
        # Lakukan: 2 cascade one2many dengan _auto_join, tes akhir daun adalah id
        diri._reinit_mock()
        partner_ids = partner_obj.search(cr, uid, ['&', (1, '=', 1), ('child_ids.bank_ids.id', 'in', [b_aa, b_ba])])
        # Hasil tes: setidaknya satu dari data kami yang ditambahkan
        self.assertTrue(set([p_a]).issubset(set(partner_ids)),
            "_auto_join di one2many dengan domain hasil yang salah")
        self.assertTrue(set([p_ab, p_ba]) tidak di set(partner_ids),
            "_auto_join di one2many dengan domain hasil yang salah")
        # Uji kueri yang dihasilkan yang disajikan secara efektif oleh domain
        sql_query = self.query_list[0].get_sql()
        
        diharapkan = "%s::teks seperti %s" % (unaccent('"res_partner__child_ids__bank_ids"."acc_number"'), unaccent('%s'))
        self.assertIn(diharapkan, sql_query[1],
            "_auto_join di one2many dengan domain hasil yang salah")
        # TDE TODO: periksa domain pertama memiliki nama tabel yang benar
        self.assertIn('"res_partner__child_ids"."name" = %s', sql_query[1],
            "_auto_join di one2many dengan domain hasil yang salah")

        partner_child_ids_col._domain = lambda self: [('name', '=', '__%s' % self._name)]
        diri._reinit_mock()
        partner_ids = partner_obj.search(cr, uid, ['&', (1, '=', 1), ('child_ids.bank_ids.id', 'in', [b_aa, b_ba])])
        # Hasil tes: tidak ada
        self.assertFalse(partner_id,
            "_auto_join di one2many dengan domain hasil yang salah")

        # ----------------------------------------
        # Test5: tes berbasis hasil
        # ----------------------------------------

        partner_bank_ids_col._auto_join = Salah
        partner_child_ids_col._auto_join = Salah
        partner_state_id_col._auto_join = Salah
        partner_parent_id_col._auto_join = Salah
        state_country_id_col._auto_join = Salah
        partner_child_ids_col._domain = []
        partner_bank_ids_col._domain = []

        # Lakukan: ('child_ids.state_id.country_id.code', 'like', '..') tanpa _auto_join
        diri._reinit_mock()
        partner_ids = partner_obj.search(cr, uid, [('child_ids.state_id.country_id.code', 'like', name_test)])
        # Hasil tes: setidaknya data tambahan + data demo kami
        self.assertTrue(set([p_a, p_b]).issubset(set(partner_ids)),
            "_auto_join off: ('child_ids.state_id.country_id.code', 'like', '..') hasil salah")
        # Uji kueri yang dihasilkan
        self.assertEqual(len(self.query_list), 5,
            "_auto_join off: ('child_ids.state_id.country_id.code', 'like', '..') jumlah kueri salah")

        # Lakukan: ('child_ids.state_id.country_id.code', 'like', '..') dengan _auto_join
        partner_child_ids_col._auto_join = Benar
        partner_state_id_col._auto_join = Benar
        state_country_id_col._auto_join = Benar
        diri._reinit_mock()
        partner_ids = partner_obj.search(cr, uid, [('child_ids.state_id.country_id.code', 'like', name_test)])
        # Hasil tes: setidaknya data tambahan + data demo kami
        self.assertTrue(set([p_a, p_b]).issubset(set(partner_ids)),
            "_auto_join on: ('child_ids.state_id.country_id.code', 'like', '..') salah hasil")
        # Uji kueri yang dihasilkan
        self.assertEqual(len(self.query_list), 1,
            "_auto_join on: ('child_ids.state_id.country_id.code', 'like', '..') jumlah kueri salah")

        # Hapus ejekan dan modifikasi
        partner_bank_ids_col._auto_join = Salah
        partner_child_ids_col._auto_join = Salah
        partner_state_id_col._auto_join = Salah
        partner_parent_id_col._auto_join = Salah
        state_country_id_col._auto_join = Salah
        
    def test_40_negating_long_expression(mandiri):
        sumber = ['!','&',('user_id','=',4),('partner_id','in',[1,2])]
        harapkan = ['|',('user_id','!=',4),('partner_id','not in',[1,2])]
        self.assertEqual(expression.distribute_not(source), harapkan,
            "distribute_not pada ekspresi yang salah diterapkan")

        pos_leaves = [[('a', 'in', [])], [('d', '!=', 3)]]
        neg_leaves = [[('a', 'tidak ada', [])], [('d', '=', 3)]]

        sumber = ekspresi.ATAU([ekspresi.AND(pos_leaves)] * 1000)
        harapkan = sumber
        self.assertEqual(expression.distribute_not(source), harapkan,
            "distribute_not pada ekspresi panjang tanpa operator negasi tidak boleh mengubahnya")

        sumber = ['!'] + sumber
        harapkan = ekspresi.AND([ekspresi.OR(neg_leaves)] * 1000)
        self.assertEqual(expression.distribute_not(source), harapkan,
            "distribute_not pada ekspresi panjang salah diterapkan")
kami
    def test_30_normalize_domain(mandiri):
        ekspresi = openerp.osv.expression
        norm_domain = domain = ['&', (1, '=', 1), ('a', '=', 'b')]
        tegaskan norm_domain == ekspresi.normalisasi_domain(domain), "Domain yang dinormalisasi harus dibiarkan tidak tersentuh"
        domain = [('x', 'in', ['y', 'z']), ('a.v', '=', 'e'), '|', '|', ('a ', '=', 'b'), '!', ('c', '>', 'd'), ('e', '!=', 'f'), ('g', ' =', 'h')]
        norm_domain = ['&', '&', '&'] + domain
        tegaskan norm_domain == ekspresi.normalisasi_domain(domain), "Domain yang tidak dinormalisasi harus dinormalisasi dengan benar"
        
    def test_translate_search(mandiri):
        Negara = self.registry('res.country')
        be = self.ref('base.be')
        domain = [
            [('nama', '=', 'Belgia')],
            [('nama', 'ilike', 'Belgi')],
            [('name', 'in', ['Belgia', 'Care Bears'])],
        ]

        untuk domain di domain:
            id = Country.search(self.cr, self.uid, domain)
            self.assertListEqual([menjadi], id)

    def test_long_table_alias(sendiri):
        # Untuk menguji batas 64 karakter untuk alias tabel di PostgreSQL
        self.patch_order('res.users', 'partner_id')
        self.patch_order('res.partner', 'commercial_partner_id,company_id,name')
        self.patch_order('res.company', 'parent_id')
        self.env['res.users'].search([('name', '=', 'test')])

jika __name__ == '__main__':
    unittest2.main()
