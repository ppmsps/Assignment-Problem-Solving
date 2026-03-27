import streamlit as st

# --- 1. Data Structures ---

class MovieNode:
    def __init__(self, name, price, showtime):
        self.movie_name = name
        self.price = price
        self.showtimes = [showtime]  # เก็บเป็น List ของรอบฉาย
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
        """ค้นหาหนังด้วย Linear Search"""
        current = self.head
        while current:
            if current.movie_name == name:
                return current
            current = current.next
        return None

    def remove_movie(self, name):
        """ลบหนังออกจาก Linked List"""
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

if 'cinema' not in st.session_state:
    st.session_state.cinema = CinemaLogic()
if 'page' not in st.session_state:
    st.session_state.page = "home"
if 'selected_movie' not in st.session_state:
    st.session_state.selected_movie = None

cinema = st.session_state.cinema

# --- 3. UI Functions ---

def go_to_detail(movie_name):
    st.session_state.selected_movie = movie_name
    st.session_state.page = "detail"
    st.rerun()

def go_home():
    st.session_state.page = "home"
    st.rerun()

# --- 4. Page Routing ---

# --- PAGE: HOME ---
if st.session_state.page == "home":
    st.title("🎬 ระบบจัดการโรงภาพยนตร์")
    
    # ส่วนที่ 1: เพิ่มหนังใหม่
    with st.expander("➕ เพิ่มภาพยนตร์เรื่องใหม่"):
        col1, col2, col3 = st.columns(3)
        new_name = col1.text_input("ชื่อหนัง")
        new_price = col2.number_input("ราคาบัตร (บาท)", min_value=0, value=150)
        new_time = col3.text_input("รอบฉาย (เช่น 14:00)")
        
        if st.button("บันทึกข้อมูล"):
            if new_name and new_time:
                cinema.add_movie(new_name, new_price, new_time)
                st.success(f"เพิ่ม {new_name} เรียบร้อยแล้ว")
                st.rerun()
            else:
                st.error("กรุณากรอกข้อมูลให้ครบถ้วน")

    st.divider()

    # ส่วนที่ 2: ค้นหาและแสดงรายชื่อหนัง
    search_q = st.text_input("🔍 ค้นหาชื่อหนังในระบบ")
    all_movies = cinema.get_all_movies()
    
    st.subheader("รายชื่อหนังทั้งหมด")
    if not all_movies:
        st.info("ยังไม่มีข้อมูลหนังในระบบ")
    else:
        for m in all_movies:
            # Linear Search Filter
            if search_q.lower() in m.movie_name.lower():
                with st.container(border=True):
                    c1, c2, c3 = st.columns([2, 2, 1])
                    c1.write(f"**{m.movie_name}**")
                    c2.write(f"ราคา: {m.price} บาท | รอบฉาย: {', '.join(m.showtimes)}")
                    if c3.button("จัดการหนัง", key=f"manage_{m.movie_name}"):
                        go_to_detail(m.movie_name)

# --- PAGE: DETAIL ---
elif st.session_state.page == "detail":
    m = cinema.get_movie(st.session_state.selected_movie)
    if not m:
        st.error("ไม่พบข้อมูลหนัง")
        if st.button("กลับหน้าหลัก"): go_home()
    else:
        st.button("⬅️ กลับหน้าหลัก", on_click=go_home)
        st.title(f"🎥 จัดการหนัง: {m.movie_name}")
        
        tab1, tab2 = st.tabs(["⚙️ แก้ไขข้อมูล & รอบฉาย", "💺 จัดการที่นั่ง"])
        
        with tab1:
            col_a, col_b = st.columns(2)
            # แก้ไขราคา
            new_p = col_a.number_input("แก้ไขราคาบัตร", value=float(m.price))
            if col_a.button("อัปเดตราคา"):
                m.price = new_p
                st.success("อัปเดตราคาแล้ว")

            # จัดการรอบฉาย
            st.write("---")
            col_t1, col_t2 = st.columns(2)
            add_t = col_t1.text_input("เพิ่มรอบฉายใหม่")
            if col_t1.button("เพิ่มรอบ"):
                if add_t:
                    m.showtimes.append(add_t)
                    st.rerun()
            
            del_t = col_t2.selectbox("ลบรอบฉาย", m.showtimes)
            if col_t2.button("ยืนยันลบรอบ"):
                if len(m.showtimes) > 1:
                    m.showtimes.remove(del_t)
                    st.rerun()
                else:
                    st.warning("ต้องมีอย่างน้อย 1 รอบฉาย")

            st.write("---")
            if st.button("🗑️ ลบหนังเรื่องนี้ออกจากระบบ", type="primary"):
                if cinema.remove_movie(m.movie_name):
                    go_home()

        with tab2:
            st.subheader("ผังที่นั่ง")
            # ค้นหาที่นั่งพัง (Linear Search)
            broken = [f"({r},{c})" for r in range(6) for c in range(10) if m.seats[r][c] == 2]
            if broken:
                st.warning(f"⚠️ ที่นั่งชำรุด: {', '.join(broken)}")

            for r in range(6):
                cols = st.columns(10)
                for c in range(10):
                    status = m.seats[r][c]
                    icon = "🔴" if status == 1 else "⚠️" if status == 2 else "⬜"
                    disabled = (status == 1) # ล็อกที่จองแล้ว
                    
                    if cols[c].button(f"{icon}\n{r},{c}", key=f"s_{r}_{c}", disabled=disabled):
                        # สลับสถานะเฉพาะที่นั่งปกติ <-> พัง
                        m.seats[r][c] = 2 if status == 0 else 0
                        st.rerun()
