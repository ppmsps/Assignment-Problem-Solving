import streamlit as st

# --- 1. Data Structures ---

class MovieNode:
    def __init__(self, name, price, showtime):
        self.movie_name = name
        self.price = price
        self.showtimes = [showtime]
        # ผังที่นั่ง 2D Array (0:ว่าง, 1:จอง, 2:พัง)
        self.seats = [[0 for _ in range(10)] for _ in range(6)]
        self.next = None

class CinemaLogic:
    def __init__(self):
        self.head = None

    def add_movie(self, name, price, showtime):
        new_node = MovieNode(name, price, showtime)
        if not self.head:
            self.head = new_node
        else:
            current = self.head
            while current.next:
                current = current.next
            current.next = new_node

    def get_movie(self, name):
        current = self.head
        while current:
            if current.movie_name == name:
                return current
            current = current.next
        return None

    def remove_movie(self, name):
        current = self.head
        prev = None
        while current and current.movie_name != name:
            prev = current
            current = current.next
        if current:
            if not prev:
                self.head = current.next
            else:
                prev.next = current.next
            return True
        return False

    def get_all_movies(self):
        movies = []
        current = self.head
        while current:
            movies.append(current)
            current = current.next
        return movies

# --- 2. Streamlit Session State ---
# ใช้สำหรับจำค่าข้อมูลไว้ ไม่ให้หายไปเมื่อ Refresh หน้าจอ

if 'cinema' not in st.session_state:
    st.session_state.cinema = CinemaLogic()
if 'page' not in st.session_state:
    st.session_state.page = "home"
if 'selected_movie' not in st.session_state:
    st.session_state.selected_movie = None

# ดึงข้อมูลจาก State มาใช้งาน
cinema = st.session_state.cinema

# --- 3. UI Functions ---

def go_to_detail(movie_name):
    st.session_state.selected_movie = movie_name
    st.session_state.page = "detail"

def go_home():
    st.session_state.page = "home"
    st.session_state.selected_movie = None

# --- 4. Page Routing ---

# --- หน้าแรก: HOME ---
if st.session_state.page == "home":
    st.title("🎬 ระบบจัดการโรงภาพยนตร์")
    
    # เมนูเพิ่มหนัง
    with st.expander("➕ เพิ่มภาพยนตร์เรื่องใหม่"):
        col1, col2, col3 = st.columns([2, 1, 1])
        new_name = col1.text_input("ชื่อหนัง")
        new_price = col2.number_input("ราคา (บาท)", min_value=0, value=150)
        new_time = col3.text_input("รอบฉาย (เช่น 19:00)")
        
        if st.button("บันทึกข้อมูล", use_container_width=True):
            if new_name and new_time:
                cinema.add_movie(new_name, new_price, new_time)
                st.success(f"เพิ่ม {new_name} สำเร็จ!")
                st.rerun()
            else:
                st.error("กรุณากรอกข้อมูลให้ครบ")

    st.divider()

    # ค้นหาและแสดงผล
    search_q = st.text_input("🔍 ค้นหาชื่อหนัง")
    all_movies = cinema.get_all_movies()
    
    if not all_movies:
        st.info("ไม่มีหนังในระบบ")
    else:
        for m in all_movies:
            if search_q.lower() in m.movie_name.lower():
                with st.container(border=True):
                    c1, c2, c3 = st.columns([2, 2, 1])
                    c1.markdown(f"### {m.movie_name}")
                    c2.write(f"💰 {m.price} บาท\n\n⏰ {', '.join(m.showtimes)}")
                    # ใช้ปุ่มเรียกฟังก์ชันเปลี่ยนหน้า
                    if c3.button("จัดการ", key=f"btn_{m.movie_name}", use_container_width=True):
                        go_to_detail(m.movie_name)
                        st.rerun()

# --- หน้าจัดการ: DETAIL ---
elif st.session_state.page == "detail":
    m = cinema.get_movie(st.session_state.selected_movie)
    
    if not m:
        st.error("ไม่พบข้อมูล")
        if st.button("กลับหน้าหลัก"): 
            go_home()
            st.rerun()
    else:
        if st.button("⬅️ กลับหน้าหลัก"):
            go_home()
            st.rerun()
            
        st.title(f"🎥 {m.movie_name}")
        
        tab1, tab2 = st.tabs(["⚙️ ข้อมูล & รอบฉาย", "💺 จัดการที่นั่ง"])

        with tab1:
            # แก้ไขราคา
            new_p = st.number_input("แก้ไขราคาบัตร", value=float(m.price))
            if st.button("อัปเดตราคา"):
                m.price = new_p
                st.toast("อัปเดตราคาแล้ว!")

            st.divider()
            # รอบฉาย
            col_t1, col_t2 = st.columns(2)
            add_t = col_t1.text_input("เพิ่มรอบฉาย")
            if col_t1.button("เพิ่ม"):
                if add_t:
                    m.showtimes.append(add_t)
                    st.rerun()
            
            del_t = col_t2.selectbox("ลบรอบฉาย", m.showtimes)
            if col_t2.button("ลบออก"):
                if len(m.showtimes) > 1:
                    m.showtimes.remove(del_t)
                    st.rerun()
                else:
                    st.warning("ต้องมีอย่างน้อย 1 รอบ")

            st.divider()
            if st.button("🗑️ ลบหนังเรื่องนี้", type="primary"):
                if cinema.remove_movie(m.movie_name):
                    go_home()
                    st.rerun()

        with tab2:
            st.write("คลิกเพื่อตั้งค่า **[⚠️ ชำรุด]** หรือ **[⬜ ว่าง]**")
            
            # แสดงสถานะที่นั่งพัง
            broken_seats = [f"({r},{c})" for r in range(6) for c in range(10) if m.seats[r][c] == 2]
            if broken_seats:
                st.warning(f"ที่นั่งชำรุด: {', '.join(broken_seats)}")

            # สร้าง Matrix ที่นั่ง
            for r in range(6):
                cols = st.columns(10)
                for c in range(10):
                    status = m.seats[r][c]
                    # กำหนด icon: ⬜=ว่าง, 🔴=จอง, ⚠️=เสีย
                    icon = "🔴" if status == 1 else "⚠️" if status == 2 else "⬜"
                    
                    if cols[c].button(f"{icon}", key=f"s_{r}_{c}"):
                        # Logic: สลับสถานะ ว่าง <-> พัง (สถานะ จอง แก้ไขไม่ได้ในหน้านี้)
                        if status == 0: m.seats[r][c] = 2
                        elif status == 2: m.seats[r][c] = 0
                        st.rerun()
