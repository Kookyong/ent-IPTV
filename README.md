# ent-IPTV
단체형IPTV
import React, { useState, useEffect, useMemo, useRef } from 'react';
import { 
  Search, Plus, Edit2, Trash2, Building2, User, Phone, MapPin, 
  Calendar, FileText, Check, X, AlertCircle, Lock, Unlock, Key, 
  Hash, Download, Star, RefreshCw, Layers, CheckCircle2, Clock,
  UserPlus, LogIn, LogOut, ShieldCheck, Users, UserCheck, UserX, ChevronRight,
  Paperclip, Image, FileSpreadsheet, Upload, Eye, Share2, Copy, Globe, Smartphone
} from 'lucide-react';
import { initializeApp } from 'firebase/app';
import { getAuth, signInAnonymously, signInWithCustomToken, onAuthStateChanged } from 'firebase/auth';
import { getFirestore, doc, collection, onSnapshot, addDoc, updateDoc, deleteDoc, serverTimestamp, getDocs } from 'firebase/firestore';

// 1. Firebase Initialization (Outside Component)
const firebaseConfig = typeof __firebase_config !== 'undefined' ? JSON.parse(__firebase_config) : {};
const app = initializeApp(firebaseConfig);
const auth = getAuth(app);
const db = getFirestore(app);
const appId = typeof __app_id !== 'undefined' ? __app_id : 'default-app-id';

// 담당 직원(구분) 목록 및 업체명 목록
const STAFF_LIST = ['박국용', '박정훈', '오세웅', '서진섭', '김경민', '이근', '하형준', '손민아'];
const COMPANY_LIST = ['(주)이앤티', '㈜온파워아이앤티'];
const WORK_TYPE_LIST = ['단체형IPTV'];
const REMARKS_STATUS_LIST = ['개통예정', '개통완료', '정산완료', '보류', '취소'];
const BASE_UNIT_PRICE = 35000;
const DIFFICULT_EXTRA_PRICE = 10000;

// 엑셀 시트 기반 27개 초기 데이터 (구분 직원명, 업체명 (주)이앤티 포함, 가입번호, 공사구분/단가/난개통 포함)
const INITIAL_DATA = [
  { seq: 1, staffName: '박국용', subNumber: '500-2404-1001', company: '(주)이앤티', workType: '단체형IPTV', unitPrice: 35000, isDifficult: false, branch: '디컴스', manager: '이대운', contact: '010-6776-5602', customer: '강서K병원', address: '서울 강서구 화곡동 377-8', lines: 19, scheduledDate: '2024-04-24', remarks: '정산완료', extraNote: '', isHighlighted: false },
  { seq: 2, staffName: '박정훈', subNumber: '500-2404-1002', company: '(주)이앤티', workType: '단체형IPTV', unitPrice: 35000, isDifficult: false, branch: '주식회사 디컴스', manager: '이대운', contact: '010-6776-5602', customer: '아모레미오 클래식점(휴먼터치 1호점)', address: '수원시팔달구 향교로 3번길 36', lines: 60, scheduledDate: '2024-04-24', remarks: '정산완료', extraNote: '', isHighlighted: false },
  { seq: 3, staffName: '오세웅', subNumber: '500-2404-1003', company: '(주)이앤티', workType: '단체형IPTV', unitPrice: 35000, isDifficult: false, branch: '주식회사 디컴스', manager: '이대운', contact: '010-6776-5602', customer: '아모레미오 빈티지점(휴먼터치2호점)', address: '수원시팔달구 향교로 3번길 36', lines: 33, scheduledDate: '2024-04-25', remarks: '정산완료', extraNote: '', isHighlighted: false },
  { seq: 4, staffName: '서진섭', subNumber: '500-2404-1004', company: '㈜온파워아이앤티', workType: '단체형IPTV', unitPrice: 35000, isDifficult: false, branch: '주식회사 델컴스', manager: '이승재', contact: '010-5768-3122', customer: '인성금오요양원', address: '수원시 장안구 정조로 992번길 15-7', lines: 2, scheduledDate: '2024-04-24', remarks: '정산완료', extraNote: '', isHighlighted: false },
  { seq: 5, staffName: '김경민', subNumber: '500-2405-1005', company: '㈜온파워아이앤티', workType: '단체형IPTV', unitPrice: 35000, isDifficult: false, branch: '주식회사 디컴스', manager: '이대운', contact: '010-6776-5602', customer: '젤라', address: '서울 강북구 미아동 189-1', lines: 16, scheduledDate: '2024-05-29', remarks: '정산완료', extraNote: '', isHighlighted: false },
  { seq: 6, staffName: '이근', subNumber: '500-2405-1006', company: '(주)이앤티', workType: '단체형IPTV', unitPrice: 35000, isDifficult: false, branch: '주식회사 디컴스', manager: '이대운', contact: '010-6776-5602', customer: '신정형외과의원', address: '경기도 고양시 덕양구 행신동 616-1', lines: 9, scheduledDate: '2024-05-14', remarks: '정산완료', extraNote: '', isHighlighted: false },
  { seq: 7, staffName: '하형준', subNumber: '500-2405-1007', company: '㈜온파워아이앤티', workType: '단체형IPTV', unitPrice: 35000, isDifficult: false, branch: '㈜온파워네트웍스', manager: '', contact: '', customer: '늘사랑실버케어센터', address: '경기도 남양주시 평내동 579', lines: 1, scheduledDate: '2024-05-22', remarks: '정산완료', extraNote: '', isHighlighted: false },
  { seq: 8, staffName: '손민아', subNumber: '500-2405-1008', company: '(주)이앤티', workType: '단체형IPTV', unitPrice: 35000, isDifficult: false, branch: '㈜지디네트웍스', manager: '정산헌', contact: '010-8388-9691', customer: '세아빌1', address: '서울시 동대문구 무학로35길 19', lines: 16, scheduledDate: '2024-05-31', remarks: '정산완료', extraNote: '', isHighlighted: false },
  { seq: 9, staffName: '박국용', subNumber: '500-2405-1009', company: '(주)이앤티', workType: '단체형IPTV', unitPrice: 35000, isDifficult: false, branch: '㈜지디네트웍스', manager: '정산헌', contact: '010-8388-9691', customer: '세아빌2', address: '서울시 동대문구 천호대로13길 59-3', lines: 12, scheduledDate: '2024-05-31', remarks: '정산완료', extraNote: '', isHighlighted: false },
  { seq: 10, staffName: '박정훈', subNumber: '500-2406-1010', company: '㈜온파워아이앤티', workType: '단체형IPTV', unitPrice: 35000, isDifficult: false, branch: '주식회사 디컴스', manager: '이대운', contact: '010-6776-5602', customer: '레지던스어울림계양구청점', address: '인천광역시 계양구 오조산로57번길 11-1', lines: 54, scheduledDate: '2024-06-27', remarks: '정산완료', extraNote: '', isHighlighted: false },
  { seq: 11, staffName: '오세웅', subNumber: '500-2406-1011', company: '㈜온파워아이앤티', workType: '단체형IPTV', unitPrice: 35000, isDifficult: false, branch: '주식회사 디컴스', manager: '이대운', contact: '010-6776-5602', customer: '문스테이션호점', address: '서울 강동구 천호동 295-31', lines: 47, scheduledDate: '2024-06-27', remarks: '정산완료', extraNote: '', isHighlighted: false },
  { seq: 12, staffName: '서진섭', subNumber: '500-2406-1012', company: '(주)이앤티', workType: '단체형IPTV', unitPrice: 35000, isDifficult: false, branch: '㈜지디네트웍스', manager: '정산헌', contact: '010-8388-9691', customer: '이효국', address: '서울 강남구 개포동 1218-20', lines: 15, scheduledDate: '2024-06-28', remarks: '정산완료', extraNote: '', isHighlighted: false },
  { seq: 13, staffName: '김경민', subNumber: '500-2407-1013', company: '㈜온파워아이앤티', workType: '단체형IPTV', unitPrice: 35000, isDifficult: false, branch: '주식회사 디컴스', manager: '이대운', contact: '010-6776-5602', customer: '반포동앤드원스테이', address: '서초구 반포동 54-12 3층', lines: 32, scheduledDate: '2024-07-25', remarks: '정산완료', extraNote: '', isHighlighted: false },
  { seq: 14, staffName: '이근', subNumber: '500-2407-1014', company: '(주)이앤티', workType: '단체형IPTV', unitPrice: 35000, isDifficult: false, branch: '주식회사 디컴스', manager: '이대운', contact: '010-6776-5602', customer: '자양한방병원', address: '연수구 신송로 154 4층', lines: 40, scheduledDate: '2024-07-26', remarks: '정산완료', extraNote: '', isHighlighted: false },
  { seq: 15, staffName: '하형준', subNumber: '500-2407-1015', company: '㈜온파워아이앤티', workType: '단체형IPTV', unitPrice: 35000, isDifficult: false, branch: '주식회사 디컴스', manager: '이대운', contact: '010-6776-5602', customer: '대원장모텔', address: '인천 중구 반달로 9번길 21', lines: 16, scheduledDate: '2024-07-30', remarks: '정산완료', extraNote: '', isHighlighted: false },
  { seq: 16, staffName: '손민아', subNumber: '500-2408-1016', company: '㈜온파워아이앤티', workType: '단체형IPTV', unitPrice: 35000, isDifficult: false, branch: '비플', manager: '이병렬', contact: '010-7179-5750', customer: '한마음외과의원', address: '권선구 경수대로 321', lines: 14, scheduledDate: '2024-08-01', remarks: '정산완료', extraNote: '', isHighlighted: false },
  { seq: 17, staffName: '박국용', subNumber: '500-2408-1017', company: '(주)이앤티', workType: '단체형IPTV', unitPrice: 35000, isDifficult: false, branch: 'JD네트웍스', manager: '정산헌', contact: '010-8388-9691', customer: '싱글하우스', address: '일산로 143, 5층 (백석동1146, 형진빌딩)', lines: 28, scheduledDate: '2024-08-13', remarks: '정산완료', extraNote: '', isHighlighted: false },
  { seq: 18, staffName: '박정훈', subNumber: '500-2408-1018', company: '㈜온파워아이앤티', workType: '단체형IPTV', unitPrice: 35000, isDifficult: false, branch: '㈜온파워네트웍스', manager: '', contact: '', customer: '살렘요양원', address: '파주시 방촌로 548번길 40-3', lines: 1, scheduledDate: '2024-08-13', remarks: '정산서류제출완료', extraNote: '', isHighlighted: false },
  { seq: 19, staffName: '오세웅', subNumber: '500-2408-1019', company: '㈜온파워아이앤티', workType: '단체형IPTV', unitPrice: 35000, isDifficult: false, branch: '비플', manager: '이병렬', contact: '010-8408-0720', customer: '굿모닝노인전문요양원', address: '평택시 현덕면 현덕로 1095-5', lines: 12, scheduledDate: '2024-08-21', remarks: '정산완료', extraNote: '', isHighlighted: false },
  { seq: 20, staffName: '서진섭', subNumber: '500-2408-1020', company: '㈜온파워아이앤티', workType: '단체형IPTV', unitPrice: 35000, isDifficult: false, branch: 'JD네트웍스', manager: '정산헌', contact: '010-8388-9691', customer: '위드니스케이', address: '의정부시 녹양로 55', lines: 12, scheduledDate: '2024-08-22', remarks: '정산완료', extraNote: '', isHighlighted: false },
  { seq: 21, staffName: '김경민', subNumber: '500-2408-1021', company: '(주)이앤티', workType: '단체형IPTV', unitPrice: 35000, isDifficult: false, branch: 'JD네트웍스', manager: '정산헌', contact: '010-8388-9691', customer: '코앤리빙텔', address: '시흥시 마유로415', lines: 52, scheduledDate: '2024-08-28', remarks: '정산완료', extraNote: '', isHighlighted: false },
  { seq: 22, staffName: '이근', subNumber: '500-2408-1022', company: '㈜온파워아이앤티', workType: '단체형IPTV', unitPrice: 35000, isDifficult: false, branch: 'JD네트웍스', manager: '정산헌', contact: '010-8388-9691', customer: '솔빛원룸텔', address: '구리시 안골로49 3층, 9층', lines: 63, scheduledDate: '작업 일정 미정', remarks: '취소', extraNote: '', isHighlighted: false },
  { seq: 23, staffName: '하형준', subNumber: '500-2408-1023', company: '㈜온파워아이앤티', workType: '단체형IPTV', unitPrice: 35000, isDifficult: false, branch: 'JD네트웍스', manager: '정산헌', contact: '010-8388-9691', customer: '애플스테이', address: '부천시 원미구 심곡동 388-11 2층', lines: 20, scheduledDate: '2024-08-23', remarks: '정산완료', extraNote: '', isHighlighted: false },
  { seq: 24, staffName: '손민아', subNumber: '500-2408-1024', company: '㈜온파워아이앤티', workType: '단체형IPTV', unitPrice: 35000, isDifficult: false, branch: 'JD네트웍스', manager: '정산헌', contact: '010-8388-9691', customer: '라움스테이원룸텔', address: '금천구 시흥대로62길 18-4 1층', lines: 12, scheduledDate: '2024-08-30', remarks: '정산완료', extraNote: '', isHighlighted: false },
  { seq: 25, staffName: '박국용', subNumber: '500-2409-1025', company: '(주)이앤티', workType: '단체형IPTV', unitPrice: 35000, isDifficult: false, branch: 'JD네트웍스', manager: '정산헌', contact: '010-8388-9691', customer: '클래스원스테이', address: '구리시 수택동 438-34', lines: 27, scheduledDate: '2024-09-04', remarks: '정산완료', extraNote: '', isHighlighted: true },
  { seq: 26, staffName: '박정훈', subNumber: '500-2409-1026', company: '(주)이앤티', workType: '단체형IPTV', unitPrice: 35000, isDifficult: false, branch: '주식회사 디컴스', manager: '이대운', contact: '010-6776-5602', customer: '다나씨엠', address: '천호대로159길 13', lines: 41, scheduledDate: '2024-09-24', remarks: '정산완료', extraNote: '12월', isHighlighted: true },
  { seq: 27, staffName: '오세웅', subNumber: '500-2409-1027', company: '(주)이앤티', workType: '단체형IPTV', unitPrice: 35000, isDifficult: false, branch: '주식회사 디컴스', manager: '이대운', contact: '010-6776-5602', customer: '서울미즈병원', address: '천호대로 1043', lines: 72, scheduledDate: '2024-09-09', remarks: '정산완료', extraNote: '12월', isHighlighted: false }
];
// 초기 회원 데이터 (관리자 보안 계정 설정 및 일반사용자 테스트 데이터)
const INITIAL_MEMBERS = [
  { username: 'ent2103', password: 'ronaldono9!@', name: '대표님(총괄관리자)', department: '대표이사', phone: '010-0000-0000', role: 'admin', status: 'approved', requestedAt: '2024-04-01' },
  { username: 'park', password: '1234', name: '박국용', department: '(주)이앤티 구내공사팀', phone: '010-1111-2222', role: 'member', status: 'approved', requestedAt: '2024-04-05' },
  { username: 'lee', password: '1234', name: '이대운', department: '주식회사 디컴스', phone: '010-6776-5602', role: 'member', status: 'approved', requestedAt: '2024-04-10' },
  { username: 'jung', password: '1234', name: '정산헌', department: 'JD네트웍스', phone: '010-8388-9691', role: 'member', status: 'pending', requestedAt: '2024-09-20' },
  { username: 'kim', password: '1234', name: '김경민', department: '구내공사 시공팀', phone: '010-5555-8282', role: 'member', status: 'pending', requestedAt: '2024-09-25' }
];

const DEFAULT_FORM = {
  seq: '',
  staffName: '박국용',
  subNumber: '',
  company: '(주)이앤티',
  workType: '단체형IPTV',
  unitPrice: BASE_UNIT_PRICE,
  isDifficult: false,
  branch: '',
  manager: '',
  contact: '',
  customer: '',
  address: '',
  lines: '',
  scheduledDate: '',
  remarks: '정산완료',
  extraNote: '',
  isHighlighted: false,
  attachments: []
};

export default function App() {
  const [user, setUser] = useState(null);
  const [records, setRecords] = useState([]);
  const [members, setMembers] = useState([]);
  const [loading, setLoading] = useState(true);
  
  // App Auth & Member Approval States
  const [currentMember, setCurrentMember] = useState(null);
  const [authTab, setAuthTab] = useState('LOGIN'); // 'LOGIN' | 'SIGNUP'
  const [loginForm, setLoginForm] = useState({ username: '', password: '' });
  const [signupForm, setSignupForm] = useState({
    username: '',
    password: '',
    name: '',
    department: '구내공사 시공팀',
    phone: ''
  });
  const [authNotice, setAuthNotice] = useState({ type: '', message: '' });
  const [isMemberModalOpen, setIsMemberModalOpen] = useState(false);

  // UI & Filter States
  const [searchQuery, setSearchQuery] = useState('');
  const [statusFilter, setStatusFilter] = useState('ALL'); // 'ALL' | 'COMPLETED' | 'IN_PROGRESS' | 'CANCELED' | 'HIGHLIGHTED'
  const [companyFilter, setCompanyFilter] = useState('ALL'); // 'ALL' | '(주)이앤티' | '㈜온파워아이앤티' ...
  const [staffFilter, setStaffFilter] = useState('ALL'); // 'ALL' | '박국용' | '박정훈' ...
  const [monthFilter, setMonthFilter] = useState('ALL'); // 'ALL' | '2024-04' | '2024-05' ... | 'UNDECIDED'
  const [isModalOpen, setIsModalOpen] = useState(false);
  const [deleteConfirmId, setDeleteConfirmId] = useState(null);
  const [resetConfirmOpen, setResetConfirmOpen] = useState(false);
  const tableSectionRef = useRef(null);

  // Attachment Management States (사진, 엑셀, PDF 첨부 모달)
  const [attachmentModalRecordId, setAttachmentModalRecordId] = useState(null);
  const [attachmentNotice, setAttachmentNotice] = useState({ type: '', message: '' });
  const [uploadingFile, setUploadingFile] = useState(false);
  const [previewAttachment, setPreviewAttachment] = useState(null);

  // 직원 공유 & 멀티유저 안내 모달 상태
  const [isShareModalOpen, setIsShareModalOpen] = useState(false);
  const [copySuccess, setCopySuccess] = useState(false);
  
  // Form State
  const [formData, setFormData] = useState(DEFAULT_FORM);
  const [editingId, setEditingId] = useState(null);

  // Masking States
  const [isMasked, setIsMasked] = useState(true);
  const [showPasswordModal, setShowPasswordModal] = useState(false);
  const [passwordInput, setPasswordInput] = useState('');
  const [passwordError, setPasswordError] = useState('');

  // --- Status Classification Helpers ---
  const isCompletedRecord = (r) => {
    const rem = (r.remarks || '').trim();
    return rem === '정산완료';
  };

  const isCanceledRecord = (r) => {
    const rem = (r.remarks || '').trim();
    return rem.includes('취소') || rem.includes('보류');
  };

  const isInProgressRecord = (r) => {
    return !isCompletedRecord(r) && !isCanceledRecord(r);
  };

  const getRemarksBadgeStyle = (remarksText) => {
    const rem = (remarksText || '').trim();
    if (rem === '정산완료') return 'bg-emerald-100 text-emerald-800 border-emerald-300';
    if (rem === '개통완료') return 'bg-blue-100 text-blue-800 border-blue-300';
    if (rem === '개통예정') return 'bg-indigo-100 text-indigo-800 border-indigo-300';
    if (rem === '보류') return 'bg-amber-100 text-amber-800 border-amber-300';
    if (rem === '취소') return 'bg-red-100 text-red-700 border-red-300';
    return 'bg-slate-100 text-slate-700 border-slate-300';
  };

  // 단가 및 총 정산금액 계산 헬퍼 (기본 35,000원 + 난개통 시 10,000원 추가)
  const getEffectiveUnitPrice = (r) => {
    const base = Number(r.unitPrice) || BASE_UNIT_PRICE;
    const extra = r.isDifficult ? DIFFICULT_EXTRA_PRICE : 0;
    return base + extra;
  };

  const getRecordTotalPrice = (r) => {
    const linesCount = Number(r.lines) || 0;
    return linesCount * getEffectiveUnitPrice(r);
  };

  // 개통예정일에서 월(YYYY-MM) 추출 헬퍼
  const getRecordMonthKey = (scheduledDate) => {
    const str = String(scheduledDate || '').trim();
    const match = str.match(/^(\d{4})-(\d{2})/);
    if (match) return `${match[1]}-${match[2]}`;
    return 'UNDECIDED';
  };

  const formatMonthLabel = (monthKey) => {
    if (monthKey === 'UNDECIDED') return '일정미정/기타';
    const [year, month] = monthKey.split('-');
    return `${year}년 ${Number(month)}월`;
  };

  // --- Masking Helper Functions ---
  const maskSubNumber = (subNo) => {
    if (!subNo) return '-';
    const str = String(subNo).trim();
    if (str.includes('-')) {
      const parts = str.split('-');
      if (parts.length >= 3) {
        return `${parts[0]}-****-${parts[parts.length - 1]}`;
      }
      if (parts.length === 2) {
        return `${parts[0]}-****`;
      }
    }
    if (str.length >= 7) {
      return str.substring(0, 3) + '****' + str.substring(str.length - 3);
    }
    if (str.length > 3) {
      return str.substring(0, 2) + '***';
    }
    return '****';
  };

  const maskPhone = (phone) => {
    if (!phone) return '-';
    const str = String(phone).trim();
    const parts = str.split('-');
    if (parts.length === 3) {
      return `${parts[0]}-****-${parts[2]}`;
    }
    if (str.length > 7) {
      return str.substring(0, 3) + '-****-' + str.substring(str.length - 4);
    }
    return '****';
  };

  const maskAddress = (address) => {
    if (!address) return '-';
    const str = String(address).trim();
    const parts = str.split(' ');
    if (parts.length > 2) {
      return `${parts[0]} ${parts[1]} ****`;
    }
    return `${parts[0]} ****`;
  };

  const handleToggleMask = () => {
    if (isMasked) {
      setShowPasswordModal(true);
      setPasswordInput('');
      setPasswordError('');
    } else {
      setIsMasked(true);
    }
  };

  const handlePasswordSubmit = (e) => {
    e.preventDefault();
    if (passwordInput === 'ronaldono9!@' || passwordInput === '1234') {
      setIsMasked(false);
      setShowPasswordModal(false);
    } else {
      setPasswordError('비밀번호가 일치하지 않습니다.');
    }
  };

  // 2. Auth Effect
  useEffect(() => {
    const initAuth = async () => {
      try {
        if (typeof __initial_auth_token !== 'undefined' && __initial_auth_token) {
          await signInWithCustomToken(auth, __initial_auth_token);
        } else {
          await signInAnonymously(auth);
        }
      } catch (err) {
        console.error("Authentication failed:", err);
      }
    };
    initAuth();
    const unsubscribe = onAuthStateChanged(auth, setUser);
    return () => unsubscribe();
  }, []);

  // 3. Data Fetching & Seeding Effect (전 직원 실시간 공유를 위한 Public 공용 데이터 경로 적용)
  useEffect(() => {
    if (!user) return;

    const recordsRef = collection(db, 'artifacts', appId, 'public', 'data', 'iptv_records');
    const membersRef = collection(db, 'artifacts', appId, 'public', 'data', 'iptv_members');

    let unsubRecords = null;
    let unsubMembers = null;

    const seedAndListen = async () => {
      try {
        // Seed IPTV Records if empty
        const snapRecords = await getDocs(recordsRef);
        if (snapRecords.empty) {
          for (const item of INITIAL_DATA) {
            await addDoc(recordsRef, { ...item, createdAt: serverTimestamp() });
          }
        } else {
          snapRecords.docs.forEach(async (docSnap, idx) => {
            const data = docSnap.data();
            const seqNum = data.seq || (idx + 1);
            const updates = {};
            if (data.subNumber === undefined) {
              updates.subNumber = `500-2404-${String(1000 + Number(seqNum)).slice(-4)}`;
              updates.seq = seqNum;
            }
            if (!data.staffName) {
              updates.staffName = STAFF_LIST[(Number(seqNum) - 1) % STAFF_LIST.length];
            }
            if (!data.workType) {
              updates.workType = '단체형IPTV';
            }
            if (data.unitPrice === undefined) {
              updates.unitPrice = BASE_UNIT_PRICE;
            }
            if (data.isDifficult === undefined) {
              updates.isDifficult = false;
            }
            if (Object.keys(updates).length > 0) {
              await updateDoc(doc(db, 'artifacts', appId, 'public', 'data', 'iptv_records', docSnap.id), updates);
            }
          });
        }

        // Seed Members if empty & Migrate Admin Credentials
        const snapMembers = await getDocs(membersRef);
        if (snapMembers.empty) {
          for (const m of INITIAL_MEMBERS) {
            await addDoc(membersRef, { ...m, createdAt: serverTimestamp() });
          }
        } else {
          // 기존 DB에 구 관리자 계정(admin)이 있으면 ent2103 / ronaldono9!@ 로 자동 마이그레이션
          const hasNewAdmin = snapMembers.docs.some(d => d.data().username === 'ent2103');
          const oldAdminDoc = snapMembers.docs.find(d => d.data().username === 'admin');
          if (oldAdminDoc && !hasNewAdmin) {
            await updateDoc(doc(db, 'artifacts', appId, 'public', 'data', 'iptv_members', oldAdminDoc.id), {
              username: 'ent2103',
              password: 'ronaldono9!@',
              role: 'admin',
              status: 'approved'
            });
          } else if (!hasNewAdmin) {
            await addDoc(membersRef, { ...INITIAL_MEMBERS[0], createdAt: serverTimestamp() });
          }
        }

        // Listen to IPTV Records
        unsubRecords = onSnapshot(recordsRef, 
          (snapshot) => {
            const fetchedRecords = snapshot.docs.map((d) => ({
              id: d.id,
              ...d.data()
            }));
            fetchedRecords.sort((a, b) => {
              const seqA = Number(a.seq) || 9999;
              const seqB = Number(b.seq) || 9999;
              if (seqA !== seqB) return seqA - seqB;
              if (a.createdAt && b.createdAt) return a.createdAt.toMillis() - b.createdAt.toMillis();
              return 0;
            });
            setRecords(fetchedRecords);
            setLoading(false);
          },
          (error) => {
            console.error("Firestore records listening error:", error);
            setLoading(false);
          }
        );

        // Listen to Members
        unsubMembers = onSnapshot(membersRef,
          (snapshot) => {
            const fetchedMembers = snapshot.docs.map((d) => ({
              id: d.id,
              ...d.data()
            }));
            setMembers(fetchedMembers);
          },
          (error) => {
            console.error("Firestore members listening error:", error);
          }
        );
      } catch (err) {
        console.error("Setup error:", err);
        setLoading(false);
      }
    };

    seedAndListen();

    return () => {
      if (unsubRecords) unsubRecords();
      if (unsubMembers) unsubMembers();
    };
  }, [user]);

  // --- Member Login & Signup Handlers ---
  const handleLoginSubmit = (e) => {
    e.preventDefault();
    setAuthNotice({ type: '', message: '' });

    const uidInput = loginForm.username.trim();
    const pwInput = loginForm.password.trim();

    if (!uidInput || !pwInput) {
      setAuthNotice({ type: 'error', message: '아이디와 비밀번호를 모두 입력해주세요.' });
      return;
    }

    // 대표님(관리자) 마스터 계정 즉시 인증 지원
    if (uidInput === 'ent2103' && pwInput === 'ronaldono9!@') {
      const adminMember = members.find(m => m.username === 'ent2103') || INITIAL_MEMBERS[0];
      setCurrentMember(adminMember);
      setLoginForm({ username: '', password: '' });
      return;
    }

    const matched = members.find(m => m.username === uidInput && m.password === pwInput);
    if (!matched) {
      setAuthNotice({ type: 'error', message: '아이디 또는 비밀번호가 일치하지 않습니다.' });
      return;
    }

    if (matched.status === 'pending') {
      setAuthNotice({
        type: 'warning',
        message: '관리자 승인 대기 중인 계정입니다. 대표님 승인 완료 후 시스템 접속이 가능합니다.'
      });
      return;
    }

    if (matched.status === 'rejected') {
      setAuthNotice({
        type: 'error',
        message: '가입 승인이 반려되었거나 접근이 제한된 계정입니다. 관리자에게 문의하세요.'
      });
      return;
    }

    setCurrentMember(matched);
    setLoginForm({ username: '', password: '' });
  };

  const handleSignupSubmit = async (e) => {
    e.preventDefault();
    if (!user) return;
    setAuthNotice({ type: '', message: '' });

    const { username, password, name, department, phone } = signupForm;
    if (!username.trim() || !password.trim() || !name.trim()) {
      setAuthNotice({ type: 'error', message: '필수 항목(아이디, 비밀번호, 성명)을 모두 입력해주세요.' });
      return;
    }

    const exists = members.some(m => m.username.toLowerCase() === username.trim().toLowerCase());
    if (exists) {
      setAuthNotice({ type: 'error', message: '이미 사용 중인 아이디입니다. 다른 아이디를 입력해주세요.' });
      return;
    }

    try {
      const membersRef = collection(db, 'artifacts', appId, 'public', 'data', 'iptv_members');
      await addDoc(membersRef, {
        username: username.trim(),
        password: password.trim(),
        name: name.trim(),
        department: department.trim() || '협력사 직원',
        phone: phone.trim() || '-',
        role: 'member',
        status: 'pending',
        requestedAt: new Date().toISOString().slice(0, 10),
        createdAt: serverTimestamp()
      });

      setSignupForm({ username: '', password: '', name: '', department: '구내공사 시공팀', phone: '' });
      setAuthTab('LOGIN');
      setAuthNotice({
        type: 'success',
        message: '회원가입 신청이 접수되었습니다! 대표님(관리자) 승인 후 로그인하실 수 있습니다.'
      });
    } catch (error) {
      console.error("Signup error:", error);
      setAuthNotice({ type: 'error', message: '가입 신청 중 오류가 발생했습니다.' });
    }
  };

  const handleUpdateMemberStatus = async (memberId, newStatus, newRole = null) => {
    if (!user) return;
    try {
      const docRef = doc(db, 'artifacts', appId, 'public', 'data', 'iptv_members', memberId);
      const payload = { status: newStatus, updatedAt: serverTimestamp() };
      if (newRole) {
        payload.role = newRole;
      }
      await updateDoc(docRef, payload);
      if (currentMember && currentMember.id === memberId && newRole) {
        setCurrentMember(prev => ({ ...prev, status: newStatus, role: newRole }));
      }
    } catch (err) {
      console.error("Error updating member status:", err);
    }
  };

  const handleUpdateMemberRole = async (memberId, newRole) => {
    if (!user) return;
    try {
      const docRef = doc(db, 'artifacts', appId, 'public', 'data', 'iptv_members', memberId);
      await updateDoc(docRef, { role: newRole, updatedAt: serverTimestamp() });
      if (currentMember && currentMember.id === memberId) {
        setCurrentMember(prev => ({ ...prev, role: newRole }));
      }
    } catch (err) {
      console.error("Error updating member role:", err);
    }
  };

  const handleDeleteMember = async (memberId) => {
    if (!user) return;
    try {
      const docRef = doc(db, 'artifacts', appId, 'public', 'data', 'iptv_members', memberId);
      await deleteDoc(docRef);
    } catch (err) {
      console.error("Error deleting member:", err);
    }
  };

  // Pending Members Count
  const pendingMembersCount = useMemo(() => {
    return members.filter(m => m.status === 'pending').length;
  }, [members]);

  // 현재 로그인한 사용자의 관리자 여부 및 본인 내역 판별 헬퍼
  const isAdmin = currentMember?.role === 'admin';

  const getCleanMemberName = (rawName) => {
    if (!rawName) return '';
    // '이대운 팀장' -> '이대운', '박국용' -> '박국용' 등으로 직함이 붙어있어도 이름 매칭 지원
    return String(rawName).trim().split(' ')[0];
  };

  const isOwnedByCurrentMember = (record) => {
    if (!currentMember) return false;
    if (currentMember.role === 'admin') return true;
    const rawName = (currentMember.name || '').trim();
    const cleanName = getCleanMemberName(rawName);
    const rStaff = (record.staffName || '').trim();
    const rManager = (record.manager || '').trim();
    const rCreator = (record.createdBy || '').trim();

    return (
      (cleanName && (rStaff === cleanName || rManager === cleanName)) ||
      (rawName && (rStaff === rawName || rManager === rawName)) ||
      (currentMember.username && rCreator === currentMember.username)
    );
  };

  // 권한이 적용된 기본 레코드 셋 (관리자: 전체 / 일반사용자: 본인 내역만)
  const accessibleRecords = useMemo(() => {
    if (!currentMember) return [];
    if (isAdmin) return records;
    return records.filter(isOwnedByCurrentMember);
  }, [records, currentMember, isAdmin]);

  // 동적 업체명 리스트 추출 ((주)이앤티, ㈜온파워아이앤티 + 신규 입력 업체 포함)
  const availableCompanies = useMemo(() => {
    const set = new Set(COMPANY_LIST);
    accessibleRecords.forEach(r => {
      if (r.company && r.company.trim()) set.add(r.company.trim());
    });
    return Array.from(set);
  }, [accessibleRecords]);

  // 개통예정일 기준 월별(YYYY-MM) 목록 추출
  const availableMonths = useMemo(() => {
    const set = new Set();
    let hasUndecided = false;
    accessibleRecords.forEach(r => {
      const mKey = getRecordMonthKey(r.scheduledDate);
      if (mKey === 'UNDECIDED') {
        hasUndecided = true;
      } else {
        set.add(mKey);
      }
    });
    const sorted = Array.from(set).sort();
    if (hasUndecided) sorted.push('UNDECIDED');
    return sorted;
  }, [accessibleRecords]);

  // Filter records in memory (Rule 2 + 일반사용자 본인 내역 전용 필터 적용)
  const filteredRecords = useMemo(() => {
    return accessibleRecords.filter(r => {
      const rem = (r.remarks || '').trim();
      const matchesStatus = 
        statusFilter === 'ALL' ? true :
        statusFilter === 'COMPLETED' ? isCompletedRecord(r) :
        statusFilter === 'IN_PROGRESS' ? isInProgressRecord(r) :
        statusFilter === 'CANCELED' ? isCanceledRecord(r) :
        statusFilter === 'HIGHLIGHTED' ? Boolean(r.isHighlighted) :
        REMARKS_STATUS_LIST.includes(statusFilter) ? rem === statusFilter : true;

      if (!matchesStatus) return false;

      // 업체명 필터
      if (companyFilter !== 'ALL' && (r.company || '').trim() !== companyFilter) {
        return false;
      }

      // 구분(직원명) 필터 (관리자 전용)
      if (isAdmin && staffFilter !== 'ALL' && (r.staffName || '').trim() !== staffFilter) {
        return false;
      }

      // 개통예정일 월별 필터
      if (monthFilter !== 'ALL' && getRecordMonthKey(r.scheduledDate) !== monthFilter) {
        return false;
      }

      if (!searchQuery.trim()) return true;

      const lowerQuery = searchQuery.toLowerCase();
      return (
        (r.customer && r.customer.toLowerCase().includes(lowerQuery)) ||
        (r.manager && r.manager.toLowerCase().includes(lowerQuery)) ||
        (r.branch && r.branch.toLowerCase().includes(lowerQuery)) ||
        (r.company && r.company.toLowerCase().includes(lowerQuery)) ||
        (r.staffName && r.staffName.toLowerCase().includes(lowerQuery)) ||
        (r.workType && r.workType.toLowerCase().includes(lowerQuery)) ||
        (r.subNumber && r.subNumber.toLowerCase().includes(lowerQuery)) ||
        (r.address && r.address.toLowerCase().includes(lowerQuery))
      );
    });
  }, [accessibleRecords, searchQuery, statusFilter, companyFilter, staffFilter, monthFilter, isAdmin]);

  // Summary Metrics (일반사용자는 본인 내역 기준으로만 대시보드 집계)
  const summary = useMemo(() => {
    const baseRecords = accessibleRecords.filter(r => {
      if (companyFilter !== 'ALL' && (r.company || '').trim() !== companyFilter) return false;
      if (isAdmin && staffFilter !== 'ALL' && (r.staffName || '').trim() !== staffFilter) return false;
      if (monthFilter !== 'ALL' && getRecordMonthKey(r.scheduledDate) !== monthFilter) return false;
      return true;
    });

    const totalCount = baseRecords.length;
    const totalLines = baseRecords.reduce((acc, cur) => acc + (Number(cur.lines) || 0), 0);
    const totalAmount = baseRecords.reduce((acc, cur) => acc + getRecordTotalPrice(cur), 0);

    const completedList = baseRecords.filter(isCompletedRecord);
    const completedCount = completedList.length;
    const completedLines = completedList.reduce((acc, cur) => acc + (Number(cur.lines) || 0), 0);
    const completedAmount = completedList.reduce((acc, cur) => acc + getRecordTotalPrice(cur), 0);

    const inProgressList = baseRecords.filter(isInProgressRecord);
    const inProgressCount = inProgressList.length;
    const inProgressLines = inProgressList.reduce((acc, cur) => acc + (Number(cur.lines) || 0), 0);

    const canceledList = baseRecords.filter(isCanceledRecord);
    const canceledCount = canceledList.length;
    const canceledLines = canceledList.reduce((acc, cur) => acc + (Number(cur.lines) || 0), 0);

    const highlightedCount = baseRecords.filter(r => r.isHighlighted).length;

    return {
      totalCount, totalLines, totalAmount,
      completedCount, completedLines, completedAmount,
      inProgressCount, inProgressLines,
      canceledCount, canceledLines,
      highlightedCount
    };
  }, [accessibleRecords, companyFilter, staffFilter, monthFilter, isAdmin]);

  const handleDashboardCardClick = (filterType) => {
    setStatusFilter(filterType);
    if (tableSectionRef.current) {
      tableSectionRef.current.scrollIntoView({ behavior: 'smooth', block: 'nearest' });
    }
  };

  // Handlers
  const handleOpenModal = (record = null) => {
    if (record) {
      if (!isOwnedByCurrentMember(record)) return;
      setFormData({
        seq: record.seq || '',
        staffName: record.staffName || '박국용',
        subNumber: record.subNumber || '',
        company: record.company || '(주)이앤티',
        workType: record.workType || '단체형IPTV',
        unitPrice: record.unitPrice || BASE_UNIT_PRICE,
        isDifficult: Boolean(record.isDifficult),
        branch: record.branch || '',
        manager: record.manager || '',
        contact: record.contact || '',
        customer: record.customer || '',
        address: record.address || '',
        lines: record.lines || '',
        scheduledDate: record.scheduledDate || '',
        remarks: record.remarks || '',
        extraNote: record.extraNote || '',
        isHighlighted: Boolean(record.isHighlighted),
        attachments: Array.isArray(record.attachments) ? record.attachments : []
      });
      setEditingId(record.id);
    } else {
      const nextSeq = records.length > 0 ? Math.max(...records.map(r => Number(r.seq) || 0)) + 1 : 1;
      const defaultSubNo = `500-2409-${String(1000 + nextSeq).slice(-4)}`;
      const myCleanName = getCleanMemberName(currentMember?.name);
      const defaultStaff = !isAdmin
        ? (myCleanName || currentMember?.name || '박국용')
        : (staffFilter !== 'ALL' ? staffFilter : STAFF_LIST[(nextSeq - 1) % STAFF_LIST.length]);

      setFormData({
        ...DEFAULT_FORM,
        seq: nextSeq,
        staffName: defaultStaff,
        manager: !isAdmin ? (myCleanName || currentMember?.name || '') : '',
        company: companyFilter !== 'ALL' ? companyFilter : '(주)이앤티',
        workType: '단체형IPTV',
        unitPrice: BASE_UNIT_PRICE,
        isDifficult: false,
        subNumber: defaultSubNo,
        attachments: []
      });
      setEditingId(null);
    }
    setIsModalOpen(true);
  };

  // --- 파일 첨부 헬퍼 (사진 자동 리사이즈/압축 및 엑셀·PDF Base64 변환) ---
  const detectAttachmentCategory = (file) => {
    const name = (file.name || '').toLowerCase();
    const type = (file.type || '').toLowerCase();
    if (type.startsWith('image/') || /\.(jpg|jpeg|png|gif|webp|bmp)$/.test(name)) {
      return 'IMAGE';
    }
    if (name.endsWith('.pdf') || type === 'application/pdf') {
      return 'PDF';
    }
    if (/\.(xlsx|xls|csv)$/.test(name) || type.includes('spreadsheet') || type.includes('excel') || type.includes('csv')) {
      return 'EXCEL';
    }
    return 'OTHER';
  };

  const formatFileSize = (bytes) => {
    const num = Number(bytes) || 0;
    if (num < 1024) return `${num} B`;
    if (num < 1024 * 1024) return `${(num / 1024).toFixed(1)} KB`;
    return `${(num / (1024 * 1024)).toFixed(2)} MB`;
  };

  const convertFileToAttachmentItem = (file) => {
    return new Promise((resolve, reject) => {
      const category = detectAttachmentCategory(file);
      if (category === 'OTHER') {
        reject(new Error(`지원하지 않는 파일 형식입니다 (${file.name}). 사진(JPG/PNG), 엑셀(XLSX/XLS/CSV), PDF 파일만 첨부 가능합니다.`));
        return;
      }

      // 이미지 파일은 클라우드 저장 용량 최적화를 위해 자동 리사이즈 압축 처리
      if (category === 'IMAGE') {
        const reader = new FileReader();
        reader.onload = (ev) => {
          const img = new window.Image();
          img.onload = () => {
            const canvas = document.createElement('canvas');
            const MAX_DIM = 1200;
            let width = img.width;
            let height = img.height;
            if (width > height && width > MAX_DIM) {
              height = Math.round((height * MAX_DIM) / width);
              width = MAX_DIM;
            } else if (height > MAX_DIM) {
              width = Math.round((width * MAX_DIM) / height);
              height = MAX_DIM;
            }
            canvas.width = width;
            canvas.height = height;
            const ctx = canvas.getContext('2d');
            ctx.drawImage(img, 0, 0, width, height);
            const compressedDataUrl = canvas.toDataURL('image/jpeg', 0.78);
            const approxBytes = Math.round((compressedDataUrl.length * 3) / 4);
            resolve({
              id: `att_${Date.now()}_${Math.random().toString(36).slice(2, 7)}`,
              name: file.name,
              category: 'IMAGE',
              mimeType: 'image/jpeg',
              size: approxBytes,
              dataUrl: compressedDataUrl,
              uploadedAt: new Date().toISOString().slice(0, 16).replace('T', ' '),
              uploadedBy: currentMember?.name || '관리자'
            });
          };
          img.onerror = () => reject(new Error('이미지 파일을 읽는 중 오류가 발생했습니다.'));
          img.src = ev.target.result;
        };
        reader.onerror = () => reject(new Error('파일 읽기에 실패했습니다.'));
        reader.readAsDataURL(file);
      } else {
        // 엑셀(XLSX, XLS, CSV) 및 PDF 파일 용량 제한 체크 (문서당 1MB 제한 보호)
        if (file.size > 650 * 1024) {
          reject(new Error(`[${file.name}] 파일 크기가 너무 큽니다 (${formatFileSize(file.size)}). 클라우드 저장을 위해 엑셀/PDF 파일은 650KB 이하로 첨부해주세요.`));
          return;
        }
        const reader = new FileReader();
        reader.onload = (ev) => {
          resolve({
            id: `att_${Date.now()}_${Math.random().toString(36).slice(2, 7)}`,
            name: file.name,
            category,
            mimeType: file.type || (category === 'PDF' ? 'application/pdf' : 'application/vnd.openxmlformats-officedocument.spreadsheetml.sheet'),
            size: file.size,
            dataUrl: ev.target.result,
            uploadedAt: new Date().toISOString().slice(0, 16).replace('T', ' '),
            uploadedBy: currentMember?.name || '관리자'
          });
        };
        reader.onerror = () => reject(new Error('파일 읽기에 실패했습니다.'));
        reader.readAsDataURL(file);
      }
    });
  };

  // 관리란의 파일 첨부 모달에서 즉시 파일 업로드
  const handleDirectRecordAttachmentUpload = async (e, record) => {
    const files = Array.from(e.target.files || []);
    e.target.value = '';
    if (!files.length || !user || !record) return;
    if (!isOwnedByCurrentMember(record)) return;

    setUploadingFile(true);
    setAttachmentNotice({ type: '', message: '' });

    try {
      const existingList = Array.isArray(record.attachments) ? [...record.attachments] : [];
      const newItems = [];

      for (const file of files) {
        const item = await convertFileToAttachmentItem(file);
        newItems.push(item);
      }

      const updatedList = [...existingList, ...newItems];
      const totalDataLen = updatedList.reduce((acc, it) => acc + (it.dataUrl ? it.dataUrl.length : 0), 0);
      if (totalDataLen > 900000) {
        setAttachmentNotice({
          type: 'error',
          message: '첨부파일 총 용량이 건당 최대 한도(약 900KB)를 초과했습니다. 일부 파일을 삭제하거나 용량이 작은 파일을 첨부해주세요.'
        });
        setUploadingFile(false);
        return;
      }

      const docRef = doc(db, 'artifacts', appId, 'public', 'data', 'iptv_records', record.id);
      await updateDoc(docRef, {
        attachments: updatedList,
        updatedAt: serverTimestamp()
      });

      setAttachmentNotice({
        type: 'success',
        message: `${newItems.length}개의 파일(사진/엑셀/PDF)이 정상적으로 첨부되었습니다.`
      });
    } catch (err) {
      setAttachmentNotice({
        type: 'error',
        message: err.message || '파일 첨부 중 오류가 발생했습니다.'
      });
    } finally {
      setUploadingFile(false);
    }
  };

  // 관리란 파일 첨부 모달에서 개별 첨부파일 삭제
  const handleDeleteRecordAttachment = async (record, attId) => {
    if (!user || !record || !isOwnedByCurrentMember(record)) return;
    try {
      const updatedList = (record.attachments || []).filter(item => item.id !== attId);
      const docRef = doc(db, 'artifacts', appId, 'public', 'data', 'iptv_records', record.id);
      await updateDoc(docRef, {
        attachments: updatedList,
        updatedAt: serverTimestamp()
      });
      if (previewAttachment && previewAttachment.id === attId) {
        setPreviewAttachment(null);
      }
      setAttachmentNotice({ type: 'success', message: '첨부파일이 삭제되었습니다.' });
    } catch (err) {
      console.error('Error deleting attachment:', err);
      setAttachmentNotice({ type: 'error', message: '첨부파일 삭제 중 오류가 발생했습니다.' });
    }
  };

  // 신규 등록 / 수정 모달 내부에서 첨부파일 추가 및 삭제
  const handleFormAttachmentUpload = async (e) => {
    const files = Array.from(e.target.files || []);
    e.target.value = '';
    if (!files.length) return;

    setUploadingFile(true);
    try {
      const currentList = Array.isArray(formData.attachments) ? [...formData.attachments] : [];
      const added = [];
      for (const file of files) {
        const item = await convertFileToAttachmentItem(file);
        added.push(item);
      }
      const nextList = [...currentList, ...added];
      setFormData(prev => ({ ...prev, attachments: nextList }));
    } catch (err) {
      console.error(err);
    } finally {
      setUploadingFile(false);
    }
  };

  const handleRemoveFormAttachment = (attId) => {
    setFormData(prev => ({
      ...prev,
      attachments: (prev.attachments || []).filter(a => a.id !== attId)
    }));
  };

  // 첨부파일 다운로드 실행
  const handleDownloadAttachment = (att) => {
    if (!att || !att.dataUrl) return;
    const link = document.createElement('a');
    link.href = att.dataUrl;
    link.setAttribute('download', att.name || 'attachment');
    document.body.appendChild(link);
    link.click();
    document.body.removeChild(link);
  };

  // 현재 첨부 모달이 열려있는 대상 레코드
  const activeAttachmentRecord = useMemo(() => {
    if (!attachmentModalRecordId) return null;
    return records.find(r => r.id === attachmentModalRecordId) || null;
  }, [records, attachmentModalRecordId]);

  const handleCloseModal = () => {
    setIsModalOpen(false);
    setFormData(DEFAULT_FORM);
    setEditingId(null);
  };

  const handleChange = (e) => {
    const { name, value, type, checked } = e.target;
    setFormData(prev => ({ 
      ...prev, 
      [name]: type === 'checkbox' ? checked : value 
    }));
  };

  const handleSubmit = async (e) => {
    e.preventDefault();
    if (!user) return;

    const myCleanName = getCleanMemberName(currentMember?.name);
    const collectionRef = collection(db, 'artifacts', appId, 'public', 'data', 'iptv_records');
    const payload = {
      ...formData,
      // 일반 사용자는 본인 이름으로만 등록/수정 가능하도록 강제 고정
      staffName: !isAdmin ? (myCleanName || formData.staffName) : formData.staffName,
      seq: Number(formData.seq) || (records.length + 1),
      lines: Number(formData.lines) || 0,
      unitPrice: Number(formData.unitPrice) || BASE_UNIT_PRICE,
      isDifficult: Boolean(formData.isDifficult),
      createdBy: currentMember?.username || 'ent2103'
    };

    try {
      if (editingId) {
        const targetRecord = records.find(r => r.id === editingId);
        if (targetRecord && !isOwnedByCurrentMember(targetRecord)) return;
        const docRef = doc(db, 'artifacts', appId, 'public', 'data', 'iptv_records', editingId);
        await updateDoc(docRef, { ...payload, updatedAt: serverTimestamp() });
      } else {
        await addDoc(collectionRef, { ...payload, createdAt: serverTimestamp() });
      }
      handleCloseModal();
    } catch (error) {
      console.error("Error saving document:", error);
    }
  };

  const handleToggleHighlight = async (record) => {
    if (!user || !isOwnedByCurrentMember(record)) return;
    try {
      const docRef = doc(db, 'artifacts', appId, 'public', 'data', 'iptv_records', record.id);
      await updateDoc(docRef, { isHighlighted: !record.isHighlighted });
    } catch (error) {
      console.error("Error toggling highlight:", error);
    }
  };

  // 테이블에서 즉시 난개통(+10,000원) 체크박스를 토글할 수 있는 핸들러
  const handleToggleDifficult = async (record) => {
    if (!user || !isOwnedByCurrentMember(record)) return;
    try {
      const docRef = doc(db, 'artifacts', appId, 'public', 'data', 'iptv_records', record.id);
      await updateDoc(docRef, { isDifficult: !record.isDifficult, updatedAt: serverTimestamp() });
    } catch (error) {
      console.error("Error toggling difficult status:", error);
    }
  };

  // 테이블에서 즉시 비고(정산상태) 드롭다운을 변경할 수 있는 핸들러
  const handleQuickRemarksChange = async (record, newRemarks) => {
    if (!user || !isOwnedByCurrentMember(record)) return;
    try {
      const docRef = doc(db, 'artifacts', appId, 'public', 'data', 'iptv_records', record.id);
      await updateDoc(docRef, { remarks: newRemarks, updatedAt: serverTimestamp() });
    } catch (error) {
      console.error("Error updating remarks:", error);
    }
  };

  const handleDelete = async (id) => {
    if (!user) return;
    const targetRecord = records.find(r => r.id === id);
    if (targetRecord && !isOwnedByCurrentMember(targetRecord)) return;
    try {
      const docRef = doc(db, 'artifacts', appId, 'public', 'data', 'iptv_records', id);
      await deleteDoc(docRef);
      setDeleteConfirmId(null);
    } catch (error) {
      console.error("Error deleting document:", error);
    }
  };

  const handleResetToExcelData = async () => {
    if (!user) return;
    setLoading(true);
    setResetConfirmOpen(false);
    try {
      const collectionRef = collection(db, 'artifacts', appId, 'public', 'data', 'iptv_records');
      const snap = await getDocs(collectionRef);
      for (const d of snap.docs) {
        await deleteDoc(doc(db, 'artifacts', appId, 'public', 'data', 'iptv_records', d.id));
      }
      for (const item of INITIAL_DATA) {
        await addDoc(collectionRef, { ...item, createdAt: serverTimestamp() });
      }
    } catch (error) {
      console.error("Error resetting data:", error);
      setLoading(false);
    }
  };

  // 직원 공유용 안내 문구 복사 핸들러
  const handleCopyShareNotice = () => {
    const shareText = `[📢 (주)이앤티 단체형IPTV 구내공사 관리시스템 접속 안내]\n\n1. 공유된 웹 링크를 통해 PC 또는 스마트폰으로 접속합니다.\n2. 처음 접속하시는 직원분은 [회원가입 신청] 탭에서 본인의 실명(예: 박국용, 박정훈 등)으로 가입을 신청해주세요.\n3. 대표님(관리자) 승인 완료 후 로그인하시면 본인의 담당 공사 내역 조회·수정 및 현장 사진/엑셀/PDF 첨부가 실시간으로 연동됩니다.`;
    try {
      navigator.clipboard.writeText(shareText);
      setCopySuccess(true);
      setTimeout(() => setCopySuccess(false), 2500);
    } catch (e) {
      console.error(e);
    }
  };

  // Export CSV (현재 마스킹 상태 및 컬럼 순서 반영: 회선수와 정산금액 사이에 단가/난개통 배치)
  const handleExportCSV = () => {
    const headers = ['순번', '구내공사(업체명)', '구분(직원명)', '공사구분', '유치점', '유치자', '연락처', '고객명', '가입번호', '주소', '회선수', '기본단가', '난개통(+10,000)', '적용단가', '총정산금액', '개통예정일', '비고', '추가메모'];
    const rows = filteredRecords.map((r, index) => [
      r.seq || (index + 1),
      r.company || '',
      r.staffName || '-',
      r.workType || '단체형IPTV',
      r.branch || '',
      r.manager || '',
      isMasked ? maskPhone(r.contact) : (r.contact || ''),
      r.customer || '',
      isMasked ? maskSubNumber(r.subNumber) : (r.subNumber || ''),
      isMasked ? maskAddress(r.address) : (r.address || ''),
      r.lines || 0,
      Number(r.unitPrice) || BASE_UNIT_PRICE,
      r.isDifficult ? 'Y (+10,000)' : 'N',
      getEffectiveUnitPrice(r),
      getRecordTotalPrice(r),
      r.scheduledDate || '',
      r.remarks || '',
      r.extraNote || ''
    ]);

    const csvContent = '\uFEFF' + [headers, ...rows]
      .map(e => e.map(cell => `"${String(cell).replace(/"/g, '""')}"`).join(','))
      .join('\n');

    const blob = new Blob([csvContent], { type: 'text/csv;charset=utf-8;' });
    const url = URL.createObjectURL(blob);
    const link = document.createElement('a');
    link.href = url;
    link.setAttribute('download', `(이앤티)단체형IPTV_구내공사_리스트_${new Date().toISOString().slice(0,10)}.csv`);
    document.body.appendChild(link);
    link.click();
    document.body.removeChild(link);
  };

  // Loading State UI
  if (!user || loading) {
    return (
      <div className="flex h-screen items-center justify-center bg-slate-900">
        <div className="flex flex-col items-center text-slate-300">
          <div className="h-11 w-11 animate-spin rounded-full border-4 border-slate-700 border-t-blue-500 mb-4"></div>
          <p className="text-lg font-semibold text-white">전산 시스템 보안 및 데이터를 불러오는 중입니다...</p>
          <p className="text-xs text-slate-400 mt-1">(주)이앤티 단체형IPTV구내공사 관리시스템</p>
        </div>
      </div>
    );
  }

  // --- AUTHENTICATION GATE (로그인 & 회원가입 신청 화면) ---
  if (!currentMember) {
    return (
      <div className="min-h-screen bg-gradient-to-br from-slate-900 via-slate-800 to-blue-950 flex items-center justify-center p-4 font-sans">
        <div className="w-full max-w-4xl grid grid-cols-1 lg:grid-cols-12 bg-white rounded-2xl shadow-2xl overflow-hidden border border-slate-700/40">
          
          {/* Left Panel: System Branding & Quick Demo Access */}
          <div className="lg:col-span-5 bg-slate-900 text-white p-6 sm:p-8 flex flex-col justify-between relative overflow-hidden">
            <div>
              <div className="inline-flex items-center gap-2 px-3 py-1 rounded-full bg-blue-500/20 border border-blue-400/30 text-blue-300 text-xs font-semibold mb-5">
                <ShieldCheck className="w-4 h-4" />
                승인제 보안 전산망
              </div>
              <h1 className="text-2xl font-extrabold tracking-tight leading-snug">
                (주)이앤티 단체형IPTV<br />구내공사 관리시스템
              </h1>
              <p className="text-xs sm:text-sm text-slate-400 mt-3 leading-relaxed">
                LGU+ 아파트·빌딩·병원 단체형 광랜 및 IPTV 구내공사 개통·정산 통합 관리 프로그램입니다.
              </p>

              <div className="mt-6 space-y-2.5 text-xs text-slate-300 bg-slate-800/70 p-4 rounded-xl border border-slate-700">
                <div className="font-bold text-amber-300 flex items-center gap-1.5">
                  <Lock className="w-3.5 h-3.5" /> 회원제 및 대표 승인 안내
                </div>
                <p className="text-slate-400 leading-relaxed">
                  • 신규 직원은 <strong>[회원가입 신청]</strong> 후 대표님(관리자)의 승인이 완료되어야 로그인이 가능합니다.<br />
                  • 일반 직원은 본인 담당 공사 내역만 열람·수정 가능하며, 개인정보(가입번호·연락처·주소)는 마스킹 보호됩니다.
                </p>
              </div>
            </div>

            {/* Security Notice Footer */}
            <div className="mt-8 pt-5 border-t border-slate-800 text-xs text-slate-400 flex items-center justify-between">
              <span className="flex items-center gap-1.5">
                <ShieldCheck className="w-4 h-4 text-emerald-400" />
                인가된 임직원 전용 시스템
              </span>
              <span className="text-slate-500 font-mono">v2.5 Secure</span>
            </div>
          </div>

          {/* Right Panel: Login / Sign-up Form */}
          <div className="lg:col-span-7 p-6 sm:p-8 flex flex-col justify-center bg-white">
            {/* Tabs */}
            <div className="flex border-b border-slate-200 mb-6">
              <button
                type="button"
                onClick={() => { setAuthTab('LOGIN'); setAuthNotice({ type: '', message: '' }); }}
                className={`flex-1 pb-3 text-sm font-bold border-b-2 transition-colors flex items-center justify-center gap-2 ${
                  authTab === 'LOGIN' ? 'border-blue-600 text-blue-600' : 'border-transparent text-slate-400 hover:text-slate-700'
                }`}
              >
                <LogIn className="w-4 h-4" />
                로그인
              </button>
              <button
                type="button"
                onClick={() => { setAuthTab('SIGNUP'); setAuthNotice({ type: '', message: '' }); }}
                className={`flex-1 pb-3 text-sm font-bold border-b-2 transition-colors flex items-center justify-center gap-2 ${
                  authTab === 'SIGNUP' ? 'border-blue-600 text-blue-600' : 'border-transparent text-slate-400 hover:text-slate-700'
                }`}
              >
                <UserPlus className="w-4 h-4" />
                회원가입 신청 (관리자 승인제)
              </button>
            </div>

            {/* Alert Notice */}
            {authNotice.message && (
              <div className={`mb-5 p-3.5 rounded-xl border text-xs sm:text-sm font-medium flex items-start gap-2.5 ${
                authNotice.type === 'error' ? 'bg-red-50 border-red-200 text-red-700' :
                authNotice.type === 'warning' ? 'bg-amber-50 border-amber-300 text-amber-800' :
                'bg-emerald-50 border-emerald-200 text-emerald-800'
              }`}>
                <AlertCircle className="w-5 h-5 shrink-0 mt-0.5" />
                <div className="leading-relaxed">{authNotice.message}</div>
              </div>
            )}

            {authTab === 'LOGIN' ? (
              <form onSubmit={handleLoginSubmit} className="space-y-4">
                <div>
                  <label className="block text-xs font-bold text-slate-700 mb-1.5">아이디</label>
                  <input
                    type="text"
                    required
                    placeholder="아이디를 입력하세요"
                    value={loginForm.username}
                    onChange={(e) => setLoginForm({ ...loginForm, username: e.target.value })}
                    className="w-full px-3.5 py-2.5 text-sm border border-slate-300 rounded-xl focus:ring-2 focus:ring-blue-500 focus:border-blue-500 outline-none"
                  />
                </div>
                <div>
                  <label className="block text-xs font-bold text-slate-700 mb-1.5">비밀번호</label>
                  <input
                    type="password"
                    required
                    placeholder="비밀번호를 입력하세요"
                    value={loginForm.password}
                    onChange={(e) => setLoginForm({ ...loginForm, password: e.target.value })}
                    className="w-full px-3.5 py-2.5 text-sm border border-slate-300 rounded-xl focus:ring-2 focus:ring-blue-500 focus:border-blue-500 outline-none"
                  />
                </div>
                <button
                  type="submit"
                  className="w-full py-3 bg-slate-900 hover:bg-slate-800 text-white font-bold text-sm rounded-xl shadow-md transition-colors mt-2"
                >
                  시스템 로그인
                </button>
                <p className="text-xs text-center text-slate-500 pt-2">
                  계정이 없으신가요? 상단의 <button type="button" onClick={() => setAuthTab('SIGNUP')} className="text-blue-600 font-bold underline">회원가입 신청</button>을 진행해주세요.
                </p>
              </form>
            ) : (
              <form onSubmit={handleSignupSubmit} className="space-y-3.5">
                <div className="grid grid-cols-1 sm:grid-cols-2 gap-3.5">
                  <div>
                    <label className="block text-xs font-bold text-slate-700 mb-1">아이디 *</label>
                    <input
                      type="text"
                      required
                      placeholder="영문/숫자 아이디"
                      value={signupForm.username}
                      onChange={(e) => setSignupForm({ ...signupForm, username: e.target.value })}
                      className="w-full px-3 py-2 text-sm border border-slate-300 rounded-lg focus:ring-2 focus:ring-blue-500 outline-none"
                    />
                  </div>
                  <div>
                    <label className="block text-xs font-bold text-slate-700 mb-1">비밀번호 *</label>
                    <input
                      type="password"
                      required
                      placeholder="비밀번호 입력"
                      value={signupForm.password}
                      onChange={(e) => setSignupForm({ ...signupForm, password: e.target.value })}
                      className="w-full px-3 py-2 text-sm border border-slate-300 rounded-lg focus:ring-2 focus:ring-blue-500 outline-none"
                    />
                  </div>
                </div>

                <div className="grid grid-cols-1 sm:grid-cols-2 gap-3.5">
                  <div>
                    <label className="block text-xs font-bold text-slate-700 mb-1">이름(실명) *</label>
                    <input
                      type="text"
                      required
                      placeholder="예: 홍길동"
                      value={signupForm.name}
                      onChange={(e) => setSignupForm({ ...signupForm, name: e.target.value })}
                      className="w-full px-3 py-2 text-sm border border-slate-300 rounded-lg focus:ring-2 focus:ring-blue-500 outline-none"
                    />
                  </div>
                  <div>
                    <label className="block text-xs font-bold text-slate-700 mb-1">소속 / 직책</label>
                    <input
                      type="text"
                      placeholder="예: 디컴스 / 현장팀장"
                      value={signupForm.department}
                      onChange={(e) => setSignupForm({ ...signupForm, department: e.target.value })}
                      className="w-full px-3 py-2 text-sm border border-slate-300 rounded-lg focus:ring-2 focus:ring-blue-500 outline-none"
                    />
                  </div>
                </div>

                <div>
                  <label className="block text-xs font-bold text-slate-700 mb-1">연락처</label>
                  <input
                    type="text"
                    placeholder="010-0000-0000"
                    value={signupForm.phone}
                    onChange={(e) => setSignupForm({ ...signupForm, phone: e.target.value })}
                    className="w-full px-3 py-2 text-sm border border-slate-300 rounded-lg focus:ring-2 focus:ring-blue-500 outline-none"
                  />
                </div>

                <div className="bg-blue-50 border border-blue-200 rounded-lg p-3 text-xs text-blue-800">
                  ※ 가입 신청 즉시 <strong>[승인대기]</strong> 상태로 등록되며, 대표님(관리자)이 승인한 후부터 전산 열람 및 입력이 가능합니다.
                </div>

                <button
                  type="submit"
                  className="w-full py-3 bg-blue-600 hover:bg-blue-700 text-white font-bold text-sm rounded-xl shadow-md transition-colors"
                >
                  회원가입 승인 요청하기
                </button>
              </form>
            )}
          </div>
        </div>
      </div>
    );
  }

  return (
    <div className="min-h-screen bg-slate-100 font-sans text-slate-800 flex flex-col">
      {/* Top Navbar */}
      <header className="bg-slate-900 text-white shadow-md z-10 sticky top-0">
        <div className="max-w-screen-2xl mx-auto px-4 sm:px-6 py-3 flex flex-wrap items-center justify-between gap-3">
          <div className="flex items-center gap-3">
            <div className="p-2 bg-blue-600/20 rounded-lg border border-blue-500/30">
              <Building2 className="w-6 h-6 text-blue-400" />
            </div>
            <div>
              <h1 className="text-base sm:text-xl font-bold tracking-wide">(주)이앤티 단체형IPTV구내공사 관리시스템</h1>
              <p className="text-xs text-slate-400 hidden sm:block">LGU+ 구내통신 및 단체형 IPTV 개통·정산 통합 전산망</p>
            </div>
          </div>

          <div className="flex items-center flex-wrap gap-2 sm:gap-3">
            {/* 실시간 클라우드 동기화 표시 배지 */}
            <span className="hidden xl:inline-flex items-center gap-1.5 px-2.5 py-1 rounded-full text-xs font-semibold bg-blue-500/20 text-blue-300 border border-blue-400/30">
              <span className="w-2 h-2 rounded-full bg-emerald-400 animate-ping" />
              전사 실시간 연동중
            </span>

            {/* 직원 공유 & 접속안내 버튼 */}
            <button
              type="button"
              onClick={() => setIsShareModalOpen(true)}
              className="inline-flex items-center gap-1.5 px-3 py-1.5 rounded-lg text-xs font-bold bg-emerald-600 hover:bg-emerald-500 text-white transition-colors shadow-sm"
            >
              <Share2 className="w-3.5 h-3.5" />
              <span>직원 공유 · 접속안내</span>
            </button>

            {/* Security Masking Badge */}
            <span className={`inline-flex items-center gap-1.5 px-3 py-1 rounded-full text-xs font-semibold ${
              isMasked ? 'bg-emerald-500/20 text-emerald-300 border border-emerald-500/30' : 'bg-amber-500/20 text-amber-300 border border-amber-500/30'
            }`}>
              {isMasked ? <Lock className="w-3.5 h-3.5" /> : <Unlock className="w-3.5 h-3.5" />}
              {isMasked ? '개인정보 보호모드' : '마스킹 해제됨'}
            </span>

            {/* Admin Member Approval Button */}
            {currentMember.role === 'admin' && (
              <button
                onClick={() => setIsMemberModalOpen(true)}
                className="relative inline-flex items-center gap-1.5 px-3 py-1.5 rounded-lg text-xs font-bold bg-indigo-600 hover:bg-indigo-500 text-white transition-colors shadow-sm"
              >
                <Users className="w-4 h-4" />
                <span>회원승인 관리</span>
                {pendingMembersCount > 0 && (
                  <span className="ml-1 px-1.5 py-0.2 bg-amber-400 text-slate-950 rounded-full text-[11px] font-extrabold animate-pulse">
                    {pendingMembersCount}
                  </span>
                )}
              </button>
            )}

            {/* Current User Info & Logout */}
            <div className="flex items-center gap-2 bg-slate-800 px-3 py-1.5 rounded-lg border border-slate-700 text-xs">
              <span className={`px-1.5 py-0.5 rounded text-[10px] font-extrabold ${
                isAdmin ? 'bg-amber-400 text-slate-950' : 'bg-blue-600 text-white'
              }`}>
                {isAdmin ? '관리자 모드' : '일반 모드(본인전용)'}
              </span>
              <span className="font-bold text-blue-300">{currentMember.name}</span>
              <span className="text-slate-400 hidden md:inline">({currentMember.department})</span>
              <button
                onClick={() => {
                  setCurrentMember(null);
                  setStaffFilter('ALL');
                  setCompanyFilter('ALL');
                  setMonthFilter('ALL');
                  setStatusFilter('ALL');
                }}
                className="ml-1 text-slate-400 hover:text-red-400 transition-colors flex items-center gap-1 font-medium"
                title="로그아웃"
              >
                <LogOut className="w-3.5 h-3.5" />
                <span className="hidden sm:inline">로그아웃</span>
              </button>
            </div>
          </div>
        </div>
      </header>

      {/* Main Content */}
      <main className="flex-1 w-full max-w-screen-2xl mx-auto p-4 sm:p-6 flex flex-col gap-5">
        
        {/* 일반 사용자(일반 모드) 접속 시 본인 내역 전용 안내 배너 */}
        {!isAdmin && (
          <div className="bg-blue-950 text-white rounded-xl px-4 py-3 shadow-sm border border-blue-800 flex flex-col sm:flex-row items-start sm:items-center justify-between gap-2">
            <div className="flex items-center gap-2.5 text-xs sm:text-sm">
              <span className="p-1.5 rounded-lg bg-blue-500 text-white font-bold">
                <UserCheck className="w-4 h-4" />
              </span>
              <span>
                현재 <strong>일반 사용자 모드({currentMember.name})</strong>로 접속 중입니다. 보안 정책에 따라 <strong>본인 담당(구분/유치자) 공사 내역({accessibleRecords.length}건)</strong>만 조회 및 수정할 수 있습니다.
              </span>
            </div>
            <span className="text-[11px] bg-blue-900/90 text-blue-200 px-2.5 py-1 rounded-md border border-blue-700 shrink-0">
              타 직원 내역 열람 제한됨
            </span>
          </div>
        )}

        {/* Pending Member Notice Banner for Admin */}
        {currentMember.role === 'admin' && pendingMembersCount > 0 && (
          <div className="bg-indigo-950 text-white rounded-xl px-4 py-3 shadow-sm border border-indigo-700 flex flex-col sm:flex-row items-start sm:items-center justify-between gap-3">
            <div className="flex items-center gap-2.5 text-xs sm:text-sm">
              <span className="p-1.5 rounded-lg bg-amber-400 text-slate-950 font-bold">
                <UserPlus className="w-4 h-4" />
              </span>
              <span>
                현재 <strong>{pendingMembersCount}명</strong>의 신규 회원가입 승인 요청이 대기 중입니다. 승인 후 해당 직원이 전산에 접속할 수 있습니다.
              </span>
            </div>
            <button
              onClick={() => setIsMemberModalOpen(true)}
              className="px-3.5 py-1.5 bg-amber-400 hover:bg-amber-300 text-slate-950 font-bold text-xs rounded-lg transition-colors shrink-0"
            >
              승인 내역 확인하기 →
            </button>
          </div>
        )}

        {/* Interactive Summary Dashboard Cards (클릭 시 아래 리스트 즉시 필터링) */}
        <div>
          <div className="flex items-center justify-between mb-2">
            <p className="text-xs font-bold text-slate-500 flex items-center gap-1.5">
              📊 대시보드 항목을 클릭하면 아래에 해당 공사 리스트만 즉시 분류되어 나타납니다.
            </p>
            {statusFilter !== 'ALL' && (
              <button
                onClick={() => setStatusFilter('ALL')}
                className="text-xs font-bold text-blue-600 hover:underline flex items-center gap-1"
              >
                <RefreshCw className="w-3 h-3" /> 전체 리스트 다시 보기
              </button>
            )}
          </div>

          <div className="grid grid-cols-2 lg:grid-cols-4 gap-4">
            {/* Card 1: ALL */}
            <button
              type="button"
              onClick={() => handleDashboardCardClick('ALL')}
              className={`text-left bg-white rounded-xl p-4 shadow-sm border transition-all flex items-center justify-between ${
                statusFilter === 'ALL'
                  ? 'border-blue-600 ring-2 ring-blue-500/20 bg-blue-50/20'
                  : 'border-slate-200 hover:border-blue-300 hover:shadow'
              }`}
            >
              <div>
                <p className="text-xs font-bold text-slate-500">전체 구내공사 리스트</p>
                <p className="text-xl sm:text-2xl font-extrabold text-slate-900 mt-1">
                  {summary.totalCount}<span className="text-xs sm:text-sm font-normal text-slate-500 ml-1">건</span>
                </p>
                <p className="text-[11px] font-semibold text-blue-600 mt-1">
                  총 {summary.totalLines.toLocaleString()} 회선 · {summary.totalAmount.toLocaleString()}원
                </p>
              </div>
              <div className="p-3 bg-slate-100 rounded-xl text-slate-700">
                <Layers className="w-6 h-6" />
              </div>
            </button>

            {/* Card 2: COMPLETED */}
            <button
              type="button"
              onClick={() => handleDashboardCardClick('COMPLETED')}
              className={`text-left bg-white rounded-xl p-4 shadow-sm border transition-all flex items-center justify-between ${
                statusFilter === 'COMPLETED'
                  ? 'border-emerald-600 ring-2 ring-emerald-500/20 bg-emerald-50/20'
                  : 'border-slate-200 hover:border-emerald-300 hover:shadow'
              }`}
            >
              <div>
                <p className="text-xs font-bold text-slate-500">정산완료 내역</p>
                <p className="text-xl sm:text-2xl font-extrabold text-emerald-600 mt-1">
                  {summary.completedCount}<span className="text-xs sm:text-sm font-normal text-slate-500 ml-1">건</span>
                </p>
                <p className="text-[11px] font-semibold text-emerald-700 mt-1">
                  개통 {summary.completedLines.toLocaleString()} 회선 · {summary.completedAmount.toLocaleString()}원
                </p>
              </div>
              <div className="p-3 bg-emerald-50 rounded-xl text-emerald-600">
                <CheckCircle2 className="w-6 h-6" />
              </div>
            </button>

            {/* Card 3: IN_PROGRESS */}
            <button
              type="button"
              onClick={() => handleDashboardCardClick('IN_PROGRESS')}
              className={`text-left bg-white rounded-xl p-4 shadow-sm border transition-all flex items-center justify-between ${
                statusFilter === 'IN_PROGRESS'
                  ? 'border-amber-500 ring-2 ring-amber-500/20 bg-amber-50/20'
                  : 'border-slate-200 hover:border-amber-300 hover:shadow'
              }`}
            >
              <div>
                <p className="text-xs font-bold text-slate-500">진행중 / 서류제출</p>
                <p className="text-xl sm:text-2xl font-extrabold text-amber-600 mt-1">
                  {summary.inProgressCount}<span className="text-xs sm:text-sm font-normal text-slate-500 ml-1">건</span>
                </p>
                <p className="text-[11px] font-semibold text-amber-700 mt-1">
                  대기 {summary.inProgressLines.toLocaleString()} 회선
                </p>
              </div>
              <div className="p-3 bg-amber-50 rounded-xl text-amber-600">
                <Clock className="w-6 h-6" />
              </div>
            </button>

            {/* Card 4: CANCELED */}
            <button
              type="button"
              onClick={() => handleDashboardCardClick('CANCELED')}
              className={`text-left bg-white rounded-xl p-4 shadow-sm border transition-all flex items-center justify-between ${
                statusFilter === 'CANCELED'
                  ? 'border-red-500 ring-2 ring-red-500/20 bg-red-50/20'
                  : 'border-slate-200 hover:border-red-300 hover:shadow'
              }`}
            >
              <div>
                <p className="text-xs font-bold text-slate-500">취소 / 일정미정</p>
                <p className="text-xl sm:text-2xl font-extrabold text-red-500 mt-1">
                  {summary.canceledCount}<span className="text-xs sm:text-sm font-normal text-slate-500 ml-1">건</span>
                </p>
                <p className="text-[11px] font-semibold text-red-600 mt-1">
                  보류 {summary.canceledLines.toLocaleString()} 회선
                </p>
              </div>
              <div className="p-3 bg-red-50 rounded-xl text-red-500">
                <AlertCircle className="w-6 h-6" />
              </div>
            </button>
          </div>
        </div>

        {/* Control Bar (Search, Company Filter, Staff Filter & Actions) */}
        <div ref={tableSectionRef} className="bg-white rounded-xl shadow-sm border border-slate-200 p-4 flex flex-col gap-3.5">
          {/* Top Row: Company Quick Tabs & Staff Filter */}
          <div className="flex flex-wrap items-center justify-between gap-3 pb-3 border-b border-slate-100">
            {/* 업체명 필터 탭 ((주)이앤티 / ㈜온파워아이앤티 등) */}
            <div className="flex flex-wrap items-center gap-1.5">
              <span className="text-xs font-extrabold text-slate-600 mr-1 flex items-center gap-1">
                <Building2 className="w-3.5 h-3.5 text-blue-600" /> 업체명 필터:
              </span>
              <button
                type="button"
                onClick={() => setCompanyFilter('ALL')}
                className={`px-3 py-1.5 rounded-lg text-xs font-bold transition-all ${
                  companyFilter === 'ALL'
                    ? 'bg-slate-900 text-white shadow-sm'
                    : 'bg-slate-100 text-slate-600 hover:bg-slate-200'
                }`}
              >
                전체 업체 ({accessibleRecords.length})
              </button>
              {availableCompanies.map((comp) => {
                const count = accessibleRecords.filter(r => (r.company || '').trim() === comp).length;
                const isENT = comp.includes('이앤티');
                return (
                  <button
                    key={comp}
                    type="button"
                    onClick={() => setCompanyFilter(comp)}
                    className={`px-3 py-1.5 rounded-lg text-xs font-bold transition-all flex items-center gap-1.5 ${
                      companyFilter === comp
                        ? isENT
                          ? 'bg-blue-600 text-white shadow-sm ring-2 ring-blue-300'
                          : 'bg-indigo-600 text-white shadow-sm ring-2 ring-indigo-300'
                        : isENT
                        ? 'bg-blue-50 text-blue-700 border border-blue-200 hover:bg-blue-100'
                        : 'bg-slate-100 text-slate-700 hover:bg-slate-200'
                    }`}
                  >
                    <span>{comp}</span>
                    <span className={`px-1.5 py-0.2 rounded-full text-[10px] ${
                      companyFilter === comp ? 'bg-white/20 text-white' : 'bg-white text-slate-600'
                    }`}>
                      {count}
                    </span>
                  </button>
                );
              })}
            </div>

            {/* 구분 (담당 직원명) 및 개통예정일(월별) 필터 드롭다운 */}
            <div className="flex flex-wrap items-center gap-2">
              <label className="text-xs font-extrabold text-slate-600 flex items-center gap-1">
                <User className="w-3.5 h-3.5 text-indigo-600" /> 구분(직원명):
              </label>
              {isAdmin ? (
                <select
                  value={staffFilter}
                  onChange={(e) => setStaffFilter(e.target.value)}
                  className={`px-3 py-1.5 rounded-lg text-xs font-bold border outline-none transition-colors cursor-pointer ${
                    staffFilter !== 'ALL'
                      ? 'bg-indigo-600 text-white border-indigo-600'
                      : 'bg-slate-50 text-slate-800 border-slate-300 hover:border-slate-400'
                  }`}
                >
                  <option value="ALL" className="bg-white text-slate-800">전체 직원 ({STAFF_LIST.length}명)</option>
                  {STAFF_LIST.map((staff) => {
                    const cnt = accessibleRecords.filter(r => r.staffName === staff).length;
                    return (
                      <option key={staff} value={staff} className="bg-white text-slate-800">
                        {staff} ({cnt}건)
                      </option>
                    );
                  })}
                </select>
              ) : (
                <span className="px-3 py-1.5 rounded-lg text-xs font-bold bg-indigo-100 text-indigo-900 border border-indigo-200 inline-flex items-center gap-1">
                  <Lock className="w-3 h-3 text-indigo-600" />
                  {currentMember.name} 전용 ({accessibleRecords.length}건)
                </span>
              )}

              <label className="text-xs font-extrabold text-slate-600 flex items-center gap-1 ml-1">
                <Calendar className="w-3.5 h-3.5 text-emerald-600" /> 개통예정월:
              </label>
              <select
                value={monthFilter}
                onChange={(e) => setMonthFilter(e.target.value)}
                className={`px-3 py-1.5 rounded-lg text-xs font-bold border outline-none transition-colors cursor-pointer ${
                  monthFilter !== 'ALL'
                    ? 'bg-emerald-600 text-white border-emerald-600'
                    : 'bg-slate-50 text-slate-800 border-slate-300 hover:border-slate-400'
                }`}
              >
                <option value="ALL" className="bg-white text-slate-800">전체 기간 ({accessibleRecords.length}건)</option>
                {availableMonths.map((mKey) => {
                  const cnt = accessibleRecords.filter(r => getRecordMonthKey(r.scheduledDate) === mKey).length;
                  return (
                    <option key={mKey} value={mKey} className="bg-white text-slate-800">
                      {formatMonthLabel(mKey)} ({cnt}건)
                    </option>
                  );
                })}
              </select>

              {(companyFilter !== 'ALL' || (isAdmin && staffFilter !== 'ALL') || monthFilter !== 'ALL') && (
                <button
                  type="button"
                  onClick={() => { setCompanyFilter('ALL'); setStaffFilter('ALL'); setMonthFilter('ALL'); }}
                  className="px-2.5 py-1.5 text-xs font-bold text-red-600 bg-red-50 hover:bg-red-100 rounded-lg transition-colors"
                >
                  조건 초기화
                </button>
              )}
            </div>
          </div>

          {/* Middle Row: 개통예정일 월별 원클릭 퀵 필터 바 */}
          <div className="flex flex-wrap items-center gap-1.5 pb-3 border-b border-slate-100">
            <span className="text-xs font-extrabold text-slate-600 mr-1 flex items-center gap-1">
              <Calendar className="w-3.5 h-3.5 text-emerald-600" /> 개통예정일(월별) 조회:
            </span>
            <button
              type="button"
              onClick={() => setMonthFilter('ALL')}
              className={`px-2.5 py-1 rounded-md text-xs font-bold transition-all ${
                monthFilter === 'ALL'
                  ? 'bg-emerald-600 text-white shadow-sm'
                  : 'bg-slate-100 text-slate-600 hover:bg-slate-200'
              }`}
            >
              전체 월 ({accessibleRecords.length})
            </button>
            {availableMonths.map((mKey) => {
              const cnt = accessibleRecords.filter(r => getRecordMonthKey(r.scheduledDate) === mKey).length;
              return (
                <button
                  key={mKey}
                  type="button"
                  onClick={() => setMonthFilter(monthFilter === mKey ? 'ALL' : mKey)}
                  className={`px-2.5 py-1 rounded-md text-xs font-bold transition-all flex items-center gap-1 ${
                    monthFilter === mKey
                      ? 'bg-emerald-600 text-white shadow-sm ring-2 ring-emerald-300'
                      : 'bg-emerald-50/70 text-emerald-900 border border-emerald-200 hover:bg-emerald-100'
                  }`}
                >
                  <span>{formatMonthLabel(mKey)}</span>
                  <span className={`px-1.5 py-0.2 rounded-full text-[10px] ${
                    monthFilter === mKey ? 'bg-white/20 text-white' : 'bg-white text-emerald-700'
                  }`}>
                    {cnt}
                  </span>
                </button>
              );
            })}
          </div>

          {/* Bottom Row: Search, Status Pills & Action Buttons */}
          <div className="flex flex-col lg:flex-row justify-between items-stretch lg:items-center gap-4">
            <div className="flex flex-col sm:flex-row items-stretch sm:items-center gap-3 flex-1">
              {/* Search Input */}
              <div className="relative flex-1 max-w-md">
                <div className="absolute inset-y-0 left-0 pl-3 flex items-center pointer-events-none">
                  <Search className="h-4 w-4 text-slate-400" />
                </div>
                <input
                  type="text"
                  placeholder="고객명, 가입번호, 직원명, 업체명, 유치점 검색..."
                  className="block w-full pl-9 pr-3 py-2 border border-slate-300 rounded-lg text-sm bg-white placeholder-slate-400 focus:outline-none focus:ring-2 focus:ring-blue-500 focus:border-blue-500 transition-colors"
                  value={searchQuery}
                  onChange={(e) => setSearchQuery(e.target.value)}
                />
              </div>

              {/* Filter Pills */}
              <div className="flex flex-wrap items-center gap-1.5 bg-slate-100 p-1 rounded-lg text-xs font-medium">
                <button
                  onClick={() => setStatusFilter('ALL')}
                  className={`px-2.5 py-1.5 rounded-md transition-colors ${statusFilter === 'ALL' ? 'bg-white text-slate-900 shadow-sm font-bold' : 'text-slate-600 hover:text-slate-900'}`}
                >
                  전체 ({summary.totalCount})
                </button>
                {REMARKS_STATUS_LIST.map((statusName) => {
                  const cnt = accessibleRecords.filter(r => {
                    if (companyFilter !== 'ALL' && (r.company || '').trim() !== companyFilter) return false;
                    if (isAdmin && staffFilter !== 'ALL' && (r.staffName || '').trim() !== staffFilter) return false;
                    if (monthFilter !== 'ALL' && getRecordMonthKey(r.scheduledDate) !== monthFilter) return false;
                    return (r.remarks || '').trim() === statusName;
                  }).length;
                  return (
                    <button
                      key={statusName}
                      onClick={() => setStatusFilter(statusName)}
                      className={`px-2.5 py-1.5 rounded-md transition-colors ${
                        statusFilter === statusName
                          ? 'bg-white text-blue-700 shadow-sm font-bold ring-1 ring-blue-300'
                          : 'text-slate-600 hover:text-slate-900'
                      }`}
                    >
                      {statusName} ({cnt})
                    </button>
                  );
                })}
                <button
                  onClick={() => setStatusFilter('HIGHLIGHTED')}
                  className={`px-2.5 py-1.5 rounded-md transition-colors ${statusFilter === 'HIGHLIGHTED' ? 'bg-yellow-200 text-slate-900 shadow-sm font-bold' : 'text-slate-600 hover:text-slate-900'}`}
                >
                  ★ 중요표시 ({summary.highlightedCount})
                </button>
              </div>
            </div>
            
            {/* Action Buttons */}
            <div className="flex flex-wrap items-center gap-2">
              <button
                onClick={handleToggleMask}
                className={`flex items-center justify-center gap-1.5 px-3.5 py-2 text-sm font-medium rounded-lg transition-colors border ${
                  isMasked 
                    ? 'bg-slate-800 text-white hover:bg-slate-700 border-slate-800' 
                    : 'bg-amber-100 text-amber-900 hover:bg-amber-200 border-amber-300'
                }`}
                title="가입번호, 연락처, 주소 마스킹 해제/적용"
              >
                {isMasked ? <Lock className="w-4 h-4" /> : <Unlock className="w-4 h-4" />}
                <span>{isMasked ? '마스킹 해제' : '마스킹 적용'}</span>
              </button>

              <button
                onClick={handleExportCSV}
                className="flex items-center justify-center gap-1.5 px-3.5 py-2 text-sm font-medium rounded-lg bg-emerald-50 text-emerald-700 border border-emerald-200 hover:bg-emerald-100 transition-colors"
                title="현재 리스트를 엑셀(CSV) 파일로 저장"
              >
                <Download className="w-4 h-4" />
                <span>엑셀(CSV) 저장</span>
              </button>

              {isAdmin && (
                <button
                  onClick={() => setResetConfirmOpen(true)}
                  className="flex items-center justify-center gap-1.5 px-3 py-2 text-sm font-medium rounded-lg bg-slate-100 text-slate-700 border border-slate-200 hover:bg-slate-200 transition-colors"
                  title="엑셀 원본 27개 데이터로 복원 (관리자 전용)"
                >
                  <RefreshCw className="w-4 h-4" />
                  <span className="hidden sm:inline">원본 복원</span>
                </button>
              )}

              <button
                onClick={() => handleOpenModal()}
                className="flex items-center justify-center gap-1.5 bg-blue-600 hover:bg-blue-700 text-white px-4 py-2 rounded-lg text-sm font-semibold shadow-sm transition-colors"
              >
                <Plus className="w-4 h-4" />
                <span>{isAdmin ? '신규 내역 등록' : '내 담당 내역 등록'}</span>
              </button>
            </div>
          </div>
        </div>

        {/* Excel-like Data Table (회선수와 정산금액 사이로 단가 35,000 / 난개통 +10,000 이동) */}
        <div className="bg-white rounded-xl shadow-sm border border-slate-300 overflow-hidden flex-1 flex flex-col">
          <div className="overflow-x-auto">
            <table className="min-w-max w-full text-left text-xs sm:text-sm whitespace-nowrap border-collapse">
              <thead className="bg-slate-200/80 text-slate-800 border-b-2 border-slate-300">
                <tr>
                  <th className="px-2.5 py-3 font-bold text-center border-r border-slate-300 w-12">No.</th>
                  <th className="px-3 py-3 font-bold text-center border-r border-slate-300">구내공사(업체명)</th>
                  <th className="px-3 py-3 font-bold text-center border-r border-slate-300 bg-indigo-50/70 text-indigo-950">
                    구분(직원명)
                  </th>
                  <th className="px-3 py-3 font-bold text-center border-r border-slate-300 bg-emerald-50/70 text-emerald-950">
                    공사구분
                  </th>
                  <th className="px-3 py-3 font-bold text-center border-r border-slate-300">유치점</th>
                  <th className="px-3 py-3 font-bold text-center border-r border-slate-300">유치자</th>
                  <th className="px-3 py-3 font-bold text-center border-r border-slate-300">
                    <div className="flex items-center justify-center gap-1">
                      연락처
                      {isMasked && <Lock className="w-3 h-3 text-slate-400" />}
                    </div>
                  </th>
                  <th className="px-3 py-3 font-bold text-center border-r border-slate-300">고객명</th>
                  <th className="px-3 py-3 font-bold text-center border-r border-slate-300 bg-blue-50/60 text-blue-950">
                    <div className="flex items-center justify-center gap-1">
                      <Hash className="w-3.5 h-3.5 text-blue-600" />
                      가입번호
                      {isMasked && <Lock className="w-3 h-3 text-slate-400 ml-0.5" />}
                    </div>
                  </th>
                  <th className="px-3 py-3 font-bold text-center border-r border-slate-300 min-w-[200px]">
                    <div className="flex items-center justify-center gap-1">
                      주소
                      {isMasked && <Lock className="w-3 h-3 text-slate-400" />}
                    </div>
                  </th>
                  <th className="px-3 py-3 font-bold text-center border-r border-slate-300 w-16">회선수</th>
                  <th className="px-3 py-3 font-bold text-center border-r border-slate-300 bg-emerald-50/40 text-slate-800">
                    단가
                  </th>
                  <th className="px-3 py-3 font-bold text-center border-r border-slate-300 bg-orange-50/70 text-orange-950">
                    난개통(+10,000)
                  </th>
                  <th className="px-3 py-3 font-bold text-center border-r border-slate-300 bg-slate-100 text-slate-900">정산금액</th>
                  <th className="px-3 py-3 font-bold text-center border-r border-slate-300">개통예정일</th>
                  <th className="px-3 py-3 font-bold text-center border-r border-slate-300">비고</th>
                  <th className="px-3 py-3 font-bold text-center min-w-[175px]">관리 (사진·엑셀·PDF)</th>
                </tr>
              </thead>
              <tbody className="divide-y divide-slate-200">
                {filteredRecords.length === 0 ? (
                  <tr>
                    <td colSpan="17" className="px-4 py-14 text-center text-slate-500">
                      <div className="flex flex-col items-center gap-2">
                        <AlertCircle className="w-8 h-8 text-slate-300" />
                        <p>조건에 맞는 공사 내역이 없습니다.</p>
                      </div>
                    </td>
                  </tr>
                ) : (
                  filteredRecords.map((record, index) => {
                    const isRowHighlighted = Boolean(record.isHighlighted);
                    const isCanceled = record.remarks && record.remarks.includes('취소');
                    const isENTCompany = (record.company || '').includes('이앤티');
                    const effectiveUnitPrice = getEffectiveUnitPrice(record);
                    const totalRowPrice = getRecordTotalPrice(record);

                    return (
                      <tr 
                        key={record.id} 
                        className={`transition-colors ${
                          isRowHighlighted 
                            ? 'bg-yellow-100/90 hover:bg-yellow-200/70' 
                            : isCanceled
                            ? 'bg-red-50/40 text-slate-400 hover:bg-red-50/80'
                            : 'hover:bg-blue-50/50 even:bg-slate-50/60'
                        }`}
                      >
                        {/* 1. No. (순번) */}
                        <td className="px-2.5 py-2.5 text-center font-medium text-slate-500 border-r border-slate-200">
                          {record.seq || (index + 1)}
                        </td>

                        {/* 2. 구내공사 (업체명) */}
                        <td className="px-3 py-2.5 text-center border-r border-slate-200">
                          <button
                            type="button"
                            onClick={() => setCompanyFilter(companyFilter === record.company ? 'ALL' : record.company)}
                            className={`inline-flex items-center px-2 py-0.5 rounded text-xs font-bold transition-colors ${
                              isENTCompany
                                ? 'bg-blue-100 text-blue-800 hover:bg-blue-200'
                                : 'bg-slate-100 text-slate-700 hover:bg-slate-200'
                            }`}
                            title="클릭 시 해당 업체만 필터링"
                          >
                            {record.company}
                          </button>
                        </td>

                        {/* 3. 구분 (담당 직원명) */}
                        <td className="px-3 py-2.5 text-center border-r border-slate-200 bg-indigo-50/20">
                          <button
                            type="button"
                            onClick={() => {
                              if (isAdmin) {
                                setStaffFilter(staffFilter === record.staffName ? 'ALL' : (record.staffName || 'ALL'));
                              }
                            }}
                            className={`inline-flex items-center px-2 py-0.5 rounded-md text-xs font-bold bg-indigo-100/80 text-indigo-900 transition-colors ${
                              isAdmin ? 'hover:bg-indigo-200 cursor-pointer' : 'cursor-default'
                            }`}
                            title={isAdmin ? '클릭 시 해당 직원 내역만 필터링' : '본인 담당 내역'}
                          >
                            {record.staffName || '미지정'}
                          </button>
                        </td>

                        {/* 4. 공사구분 (단체형IPTV) */}
                        <td className="px-3 py-2.5 text-center border-r border-slate-200 bg-emerald-50/20">
                          <span className="inline-flex items-center px-2 py-0.5 rounded text-xs font-bold bg-emerald-100 text-emerald-800">
                            {record.workType || '단체형IPTV'}
                          </span>
                        </td>

                        {/* 5. 유치점 */}
                        <td className="px-3 py-2.5 text-center text-slate-700 border-r border-slate-200">
                          {record.branch || '-'}
                        </td>

                        {/* 6. 유치자 */}
                        <td className="px-3 py-2.5 text-center font-medium text-slate-800 border-r border-slate-200">
                          {record.manager || '-'}
                        </td>

                        {/* 7. 연락처 (마스킹 대상) */}
                        <td className="px-3 py-2.5 text-center font-mono text-slate-700 border-r border-slate-200">
                          {isMasked ? maskPhone(record.contact) : (record.contact || '-')}
                        </td>

                        {/* 8. 고객명 */}
                        <td className="px-3 py-2.5 font-bold text-slate-900 border-r border-slate-200">
                          <div className="flex items-center justify-between gap-2">
                            <span>{record.customer}</span>
                            {record.extraNote && (
                              <span className="px-1.5 py-0.5 text-[10px] font-bold bg-slate-200 text-slate-700 rounded">
                                {record.extraNote}
                              </span>
                            )}
                          </div>
                        </td>

                        {/* 9. 가입번호 (고객명 바로 우측 + 마스킹 대상) */}
                        <td className="px-3 py-2.5 text-center font-mono text-xs sm:text-sm font-semibold text-blue-900 bg-blue-50/20 border-r border-slate-200">
                          {isMasked ? maskSubNumber(record.subNumber) : (record.subNumber || '-')}
                        </td>

                        {/* 10. 주소 (마스킹 대상) */}
                        <td className="px-3 py-2.5 text-slate-700 border-r border-slate-200 max-w-xs truncate" title={isMasked ? '보안 마스킹 됨' : record.address}>
                          {isMasked ? maskAddress(record.address) : record.address}
                        </td>

                        {/* 11. 회선수 */}
                        <td className="px-3 py-2.5 text-center font-bold text-slate-900 border-r border-slate-200">
                          {record.lines}
                        </td>

                        {/* 12. 단가 (회선수와 정산금액 사이로 이동: 기본 35,000원 / 난개통 체크 시 45,000원 표시) */}
                        <td className="px-3 py-2.5 text-right font-mono font-bold text-slate-800 border-r border-slate-200 bg-emerald-50/10">
                          <div>{effectiveUnitPrice.toLocaleString()}원</div>
                          {record.isDifficult && (
                            <div className="text-[10px] text-orange-600 font-semibold">
                              (기본 {(Number(record.unitPrice) || BASE_UNIT_PRICE).toLocaleString()} + 난개통 10,000)
                            </div>
                          )}
                        </td>

                        {/* 13. 난개통 체크박스 (+10,000원, 회선수와 정산금액 사이로 이동) */}
                        <td className="px-3 py-2.5 text-center border-r border-slate-200 bg-orange-50/20">
                          <label className="inline-flex items-center justify-center gap-1.5 cursor-pointer select-none px-2 py-1 rounded hover:bg-orange-100/60 transition-colors">
                            <input
                              type="checkbox"
                              checked={Boolean(record.isDifficult)}
                              onChange={() => handleToggleDifficult(record)}
                              className="w-4 h-4 text-orange-600 rounded border-slate-300 focus:ring-orange-500 cursor-pointer"
                            />
                            <span className={`text-xs font-bold ${record.isDifficult ? 'text-orange-700' : 'text-slate-400'}`}>
                              {record.isDifficult ? '+10,000' : '일반'}
                            </span>
                          </label>
                        </td>

                        {/* 14. 총 정산금액 (회선수 × 적용단가) */}
                        <td className="px-3 py-2.5 text-right font-mono font-bold text-blue-900 bg-slate-50/80 border-r border-slate-200">
                          {totalRowPrice.toLocaleString()}원
                        </td>

                        {/* 15. 개통예정일 (클릭 시 해당 월 필터링 연동) */}
                        <td className="px-3 py-2.5 text-center text-slate-700 border-r border-slate-200">
                          <button
                            type="button"
                            onClick={() => {
                              const mKey = getRecordMonthKey(record.scheduledDate);
                              setMonthFilter(monthFilter === mKey ? 'ALL' : mKey);
                            }}
                            className="px-2 py-0.5 rounded hover:bg-emerald-50 hover:text-emerald-700 font-medium transition-colors"
                            title="클릭 시 해당 개통예정월만 필터링"
                          >
                            {record.scheduledDate}
                          </button>
                        </td>

                        {/* 16. 비고 (정산상태 5종 원클릭 드롭다운 선택 지원) */}
                        <td className="px-2.5 py-2 text-center border-r border-slate-200">
                          <select
                            value={REMARKS_STATUS_LIST.includes(record.remarks) ? record.remarks : (record.remarks || '개통예정')}
                            onChange={(e) => handleQuickRemarksChange(record, e.target.value)}
                            className={`px-2.5 py-1 rounded-md text-xs font-bold border outline-none cursor-pointer transition-colors ${getRemarksBadgeStyle(record.remarks)}`}
                            title="클릭하여 정산상태(비고) 즉시 변경"
                          >
                            {!REMARKS_STATUS_LIST.includes(record.remarks) && record.remarks && (
                              <option value={record.remarks} className="bg-white text-slate-800">
                                {record.remarks}
                              </option>
                            )}
                            {REMARKS_STATUS_LIST.map((st) => (
                              <option key={st} value={st} className="bg-white text-slate-800 font-semibold">
                                {st}
                              </option>
                            ))}
                          </select>
                        </td>

                        {/* 17. 관리 버튼 (사진·엑셀·PDF 첨부 버튼 포함) */}
                        <td className="px-2.5 py-2 text-center">
                          <div className="flex items-center justify-center gap-1">
                            {/* 사진 / 엑셀 / PDF 파일 첨부 및 열람 버튼 */}
                            <button
                              type="button"
                              onClick={() => {
                                setAttachmentModalRecordId(record.id);
                                setAttachmentNotice({ type: '', message: '' });
                                setPreviewAttachment(null);
                              }}
                              className={`px-2 py-1 rounded-md text-xs font-bold flex items-center gap-1 border transition-colors ${
                                (record.attachments && record.attachments.length > 0)
                                  ? 'bg-blue-50 text-blue-700 border-blue-300 hover:bg-blue-100'
                                  : 'bg-slate-50 text-slate-600 border-slate-200 hover:bg-slate-100 hover:text-slate-900'
                              }`}
                              title="사진, 엑셀(XLSX/CSV), PDF 파일 첨부 및 다운로드"
                            >
                              <Paperclip className="w-3.5 h-3.5" />
                              <span>첨부</span>
                              {record.attachments && record.attachments.length > 0 && (
                                <span className="px-1.5 py-0.2 bg-blue-600 text-white rounded-full text-[10px] font-extrabold">
                                  {record.attachments.length}
                                </span>
                              )}
                            </button>

                            <button 
                              onClick={() => handleToggleHighlight(record)}
                              className={`p-1.5 rounded transition-colors ${
                                isRowHighlighted ? 'text-amber-500 bg-amber-50 hover:bg-amber-100' : 'text-slate-400 hover:text-amber-500 hover:bg-slate-100'
                              }`}
                              title="형광펜(중요) 표시 토글"
                            >
                              <Star className={`w-4 h-4 ${isRowHighlighted ? 'fill-amber-400' : ''}`} />
                            </button>
                            <button 
                              onClick={() => handleOpenModal(record)}
                              className="p-1.5 text-slate-500 hover:text-blue-600 hover:bg-blue-50 rounded transition-colors"
                              title="수정"
                            >
                              <Edit2 className="w-4 h-4" />
                            </button>
                            <button 
                              onClick={() => setDeleteConfirmId(record.id)}
                              className="p-1.5 text-slate-500 hover:text-red-600 hover:bg-red-50 rounded transition-colors"
                              title="삭제"
                            >
                              <Trash2 className="w-4 h-4" />
                            </button>
                          </div>
                        </td>
                      </tr>
                    );
                  })
                )}
              </tbody>
            </table>
          </div>
          <div className="bg-slate-50 px-4 py-3 border-t border-slate-200 text-xs sm:text-sm text-slate-600 flex flex-col sm:flex-row justify-between items-center gap-2">
            <div>
              💡 <strong>공사구분(단체형IPTV)</strong> 기본 단가는 <strong>35,000원</strong>이며, 표에서 <strong>[난개통]</strong> 체크 시 회선당 <strong>+10,000원(45,000원)</strong>으로 자동 합산됩니다.
            </div>
            <div className="flex items-center gap-3">
              <span>
                조회 내역: <strong className="text-slate-900">{filteredRecords.length}</strong>건
              </span>
              <span>
                총 회선: <strong className="text-blue-600">{filteredRecords.reduce((acc, r) => acc + (Number(r.lines) || 0), 0).toLocaleString()}</strong>회선
              </span>
              <span>
                총 정산금액: <strong className="text-emerald-700">{filteredRecords.reduce((acc, r) => acc + getRecordTotalPrice(r), 0).toLocaleString()}</strong>원
              </span>
            </div>
          </div>
        </div>
      </main>

      {/* Admin Member Approval Management Modal */}
      {isMemberModalOpen && (
        <div className="fixed inset-0 z-50 flex items-center justify-center p-4 bg-slate-900/60 backdrop-blur-sm overflow-y-auto">
          <div className="bg-white rounded-2xl shadow-xl w-full max-w-3xl overflow-hidden my-8 border border-slate-200">
            <div className="px-6 py-4 border-b border-slate-200 flex items-center justify-between bg-slate-900 text-white">
              <div className="flex items-center gap-2.5">
                <Users className="w-5 h-5 text-amber-400" />
                <div>
                  <h2 className="text-base sm:text-lg font-bold">회원가입 승인 및 계정 관리 (대표님 전용)</h2>
                  <p className="text-xs text-slate-400">신규 가입 신청한 직원을 승인하거나 권한을 관리합니다.</p>
                </div>
              </div>
              <button onClick={() => setIsMemberModalOpen(false)} className="text-slate-400 hover:text-white p-1 rounded-full hover:bg-slate-800">
                <X className="w-5 h-5" />
              </button>
            </div>

            <div className="p-6 max-h-[75vh] overflow-y-auto">
              <div className="overflow-x-auto border border-slate-200 rounded-xl">
                <table className="w-full text-left text-xs sm:text-sm whitespace-nowrap">
                  <thead className="bg-slate-100 text-slate-700 border-b border-slate-200">
                    <tr>
                      <th className="px-3.5 py-2.5 font-bold">상태</th>
                      <th className="px-3.5 py-2.5 font-bold">아이디</th>
                      <th className="px-3.5 py-2.5 font-bold">이름</th>
                      <th className="px-3.5 py-2.5 font-bold">소속/직책</th>
                      <th className="px-3.5 py-2.5 font-bold">연락처</th>
                      <th className="px-3.5 py-2.5 font-bold text-center">권한 등급 (모드)</th>
                      <th className="px-3.5 py-2.5 font-bold text-center">관리자 승인 처리</th>
                    </tr>
                  </thead>
                  <tbody className="divide-y divide-slate-200">
                    {members.map((m) => (
                      <tr key={m.id || m.username} className={m.status === 'pending' ? 'bg-amber-50/70' : 'hover:bg-slate-50'}>
                        <td className="px-3.5 py-3">
                          {m.status === 'approved' && (
                            <span className="inline-flex items-center gap-1 px-2 py-0.5 rounded-full text-xs font-bold bg-emerald-100 text-emerald-800">
                              <UserCheck className="w-3 h-3" /> 승인완료
                            </span>
                          )}
                          {m.status === 'pending' && (
                            <span className="inline-flex items-center gap-1 px-2 py-0.5 rounded-full text-xs font-bold bg-amber-200 text-amber-950">
                              <Clock className="w-3 h-3" /> 승인대기
                            </span>
                          )}
                          {m.status === 'rejected' && (
                            <span className="inline-flex items-center gap-1 px-2 py-0.5 rounded-full text-xs font-bold bg-red-100 text-red-700">
                              <UserX className="w-3 h-3" /> 반려/차단
                            </span>
                          )}
                        </td>
                        <td className="px-3.5 py-3 font-mono font-bold text-slate-900">{m.username}</td>
                        <td className="px-3.5 py-3 font-bold text-slate-800">
                          {m.name}
                          {m.role === 'admin' ? (
                            <span className="ml-1.5 px-1.5 py-0.5 text-[10px] bg-slate-900 text-amber-300 rounded font-bold">관리자</span>
                          ) : (
                            <span className="ml-1.5 px-1.5 py-0.5 text-[10px] bg-slate-200 text-slate-700 rounded font-bold">일반</span>
                          )}
                        </td>
                        <td className="px-3.5 py-3 text-slate-600">{m.department}</td>
                        <td className="px-3.5 py-3 font-mono text-slate-600">{m.phone}</td>
                        <td className="px-3.5 py-3 text-center">
                          {m.username === 'ent2103' ? (
                            <span className="inline-flex items-center gap-1 px-2.5 py-1 rounded-lg text-xs font-bold bg-slate-900 text-amber-300">
                              <ShieldCheck className="w-3.5 h-3.5" /> 대표 관리자
                            </span>
                          ) : (
                            <select
                              value={m.role || 'member'}
                              onChange={(e) => handleUpdateMemberRole(m.id, e.target.value)}
                              className={`px-2.5 py-1 rounded-lg text-xs font-bold border outline-none cursor-pointer transition-colors ${
                                m.role === 'admin'
                                  ? 'bg-indigo-50 text-indigo-900 border-indigo-300'
                                  : 'bg-white text-slate-700 border-slate-300'
                              }`}
                            >
                              <option value="member">일반 모드</option>
                              <option value="admin">관리자 모드</option>
                            </select>
                          )}
                        </td>
                        <td className="px-3.5 py-3 text-center">
                          {m.username === 'ent2103' ? (
                            <span className="text-xs text-slate-400 font-medium">기본 계정</span>
                          ) : (
                            <div className="flex items-center justify-center gap-1.5">
                              {m.status !== 'approved' && (
                                <>
                                  <button
                                    onClick={() => handleUpdateMemberStatus(m.id, 'approved', 'member')}
                                    className="px-2.5 py-1 bg-blue-600 hover:bg-blue-700 text-white text-xs font-bold rounded-lg transition-colors shadow-sm"
                                    title="일반 직원 권한으로 가입 승인"
                                  >
                                    일반 승인
                                  </button>
                                  <button
                                    onClick={() => handleUpdateMemberStatus(m.id, 'approved', 'admin')}
                                    className="px-2.5 py-1 bg-indigo-600 hover:bg-indigo-700 text-white text-xs font-bold rounded-lg transition-colors shadow-sm"
                                    title="관리자 모드 권한으로 가입 승인"
                                  >
                                    관리자 승인
                                  </button>
                                </>
                              )}
                              {m.status !== 'rejected' && (
                                <button
                                  onClick={() => handleUpdateMemberStatus(m.id, 'rejected')}
                                  className="px-2.5 py-1 bg-amber-100 hover:bg-amber-200 text-amber-900 text-xs font-bold rounded-lg transition-colors"
                                >
                                  {m.status === 'approved' ? '승인취소' : '반려'}
                                </button>
                              )}
                              <button
                                onClick={() => handleDeleteMember(m.id)}
                                className="p-1 text-slate-400 hover:text-red-600 rounded"
                                title="계정 삭제"
                              >
                                <Trash2 className="w-4 h-4" />
                              </button>
                            </div>
                          )}
                        </td>
                      </tr>
                    ))}
                  </tbody>
                </table>
              </div>
            </div>

            <div className="px-6 py-3 bg-slate-50 border-t border-slate-200 flex justify-between items-center text-xs text-slate-500">
              <span>💡 승인 완료 즉시 해당 직원은 로그인하여 전산 리스트를 열람할 수 있습니다.</span>
              <button
                onClick={() => setIsMemberModalOpen(false)}
                className="px-4 py-1.5 bg-slate-800 text-white font-semibold rounded-lg hover:bg-slate-700"
              >
                닫기
              </button>
            </div>
          </div>
        </div>
      )}

      {/* Add / Edit Form Modal */}
      {isModalOpen && (
        <div className="fixed inset-0 z-50 flex items-center justify-center p-4 bg-slate-900/60 backdrop-blur-sm overflow-y-auto">
          <div className="bg-white rounded-2xl shadow-xl w-full max-w-2xl overflow-hidden my-8 border border-slate-200">
            <div className="px-6 py-4 border-b border-slate-200 flex items-center justify-between bg-slate-800 text-white">
              <h2 className="text-lg font-bold flex items-center gap-2">
                {editingId ? <Edit2 className="w-5 h-5 text-blue-400" /> : <Plus className="w-5 h-5 text-blue-400" />}
                {editingId ? '구내공사 내역 수정' : '신규 구내공사 내역 등록'}
              </h2>
              <button onClick={handleCloseModal} className="text-slate-400 hover:text-white transition-colors p-1 rounded-full hover:bg-slate-700">
                <X className="w-5 h-5" />
              </button>
            </div>
            
            <form onSubmit={handleSubmit} className="p-6">
              <div className="grid grid-cols-1 md:grid-cols-3 gap-x-4 gap-y-4">
                
                {/* 1. 순번 (No.) */}
                <div className="space-y-1">
                  <label className="text-xs font-semibold text-slate-700 flex items-center gap-1">
                    No. (순번)
                  </label>
                  <input 
                    type="number" 
                    name="seq" 
                    value={formData.seq} 
                    onChange={handleChange} 
                    required
                    className="w-full px-3 py-2 text-sm border border-slate-300 rounded-lg focus:ring-2 focus:ring-blue-500 focus:border-blue-500 outline-none" 
                    placeholder="번호" 
                  />
                </div>

                {/* 2. 구내공사 (업체명 선택: (주)이앤티 기본 포함) */}
                <div className="space-y-1">
                  <label className="text-xs font-semibold text-blue-700 flex items-center gap-1">
                    <Building2 className="w-3.5 h-3.5 text-blue-600" /> 구내공사 (업체명)
                  </label>
                  <select
                    name="company"
                    value={formData.company}
                    onChange={handleChange}
                    required
                    className="w-full px-3 py-2 text-sm font-bold text-slate-800 border border-blue-300 bg-blue-50/30 rounded-lg focus:ring-2 focus:ring-blue-500 outline-none"
                  >
                    {availableCompanies.map((comp) => (
                      <option key={comp} value={comp}>{comp}</option>
                    ))}
                  </select>
                </div>

                {/* 3. 구분 (담당 직원 선택 - 일반사용자는 본인 이름으로 고정) */}
                <div className="space-y-1">
                  <label className="text-xs font-semibold text-indigo-700 flex items-center justify-between">
                    <span className="flex items-center gap-1">
                      <User className="w-3.5 h-3.5 text-indigo-600" /> 구분 (담당 직원)
                    </span>
                    {!isAdmin && (
                      <span className="text-[10px] font-bold text-indigo-600 bg-indigo-100 px-1.5 py-0.2 rounded">
                        본인 고정
                      </span>
                    )}
                  </label>
                  {isAdmin ? (
                    <select
                      name="staffName"
                      value={formData.staffName}
                      onChange={handleChange}
                      required
                      className="w-full px-3 py-2 text-sm font-bold text-indigo-950 border border-indigo-300 bg-indigo-50/40 rounded-lg focus:ring-2 focus:ring-indigo-500 outline-none"
                    >
                      {STAFF_LIST.map((staff) => (
                        <option key={staff} value={staff}>{staff}</option>
                      ))}
                    </select>
                  ) : (
                    <input
                      type="text"
                      name="staffName"
                      value={formData.staffName}
                      readOnly
                      className="w-full px-3 py-2 text-sm font-bold text-indigo-900 border border-indigo-200 bg-slate-100 rounded-lg cursor-not-allowed outline-none"
                    />
                  )}
                </div>

                {/* 4. 공사구분 (단체형IPTV) */}
                <div className="space-y-1">
                  <label className="text-xs font-semibold text-emerald-700 flex items-center gap-1">
                    <Layers className="w-3.5 h-3.5 text-emerald-600" /> 공사구분
                  </label>
                  <select
                    name="workType"
                    value={formData.workType}
                    onChange={handleChange}
                    required
                    className="w-full px-3 py-2 text-sm font-bold text-emerald-950 border border-emerald-300 bg-emerald-50/40 rounded-lg focus:ring-2 focus:ring-emerald-500 outline-none"
                  >
                    {WORK_TYPE_LIST.map((wt) => (
                      <option key={wt} value={wt}>{wt}</option>
                    ))}
                  </select>
                </div>

                {/* 5. 유치점 */}
                <div className="space-y-1">
                  <label className="text-xs font-semibold text-slate-700 flex items-center gap-1">
                    <MapPin className="w-3.5 h-3.5 text-slate-400" /> 유치점
                  </label>
                  <input 
                    type="text" 
                    name="branch" 
                    value={formData.branch} 
                    onChange={handleChange} 
                    required
                    className="w-full px-3 py-2 text-sm border border-slate-300 rounded-lg focus:ring-2 focus:ring-blue-500 outline-none" 
                    placeholder="예: 주식회사 디컴스" 
                  />
                </div>

                {/* 6. 유치자 */}
                <div className="space-y-1">
                  <label className="text-xs font-semibold text-slate-700 flex items-center gap-1">
                    <User className="w-3.5 h-3.5 text-slate-400" /> 유치자 (담당자)
                  </label>
                  <input 
                    type="text" 
                    name="manager" 
                    value={formData.manager} 
                    onChange={handleChange}
                    className="w-full px-3 py-2 text-sm border border-slate-300 rounded-lg focus:ring-2 focus:ring-blue-500 outline-none" 
                    placeholder="예: 이대운" 
                  />
                </div>

                {/* 7. 연락처 */}
                <div className="space-y-1">
                  <label className="text-xs font-semibold text-slate-700 flex items-center gap-1">
                    <Phone className="w-3.5 h-3.5 text-slate-400" /> 연락처 (마스킹 대상)
                  </label>
                  <input 
                    type="text" 
                    name="contact" 
                    value={formData.contact} 
                    onChange={handleChange}
                    className="w-full px-3 py-2 text-sm font-mono border border-slate-300 rounded-lg focus:ring-2 focus:ring-blue-500 outline-none" 
                    placeholder="010-0000-0000" 
                  />
                </div>

                {/* 8. 고객명 */}
                <div className="space-y-1">
                  <label className="text-xs font-semibold text-slate-700 flex items-center gap-1">
                    <User className="w-3.5 h-3.5 text-slate-400" /> 고객명 (설치 장소/상호)
                  </label>
                  <input 
                    type="text" 
                    name="customer" 
                    value={formData.customer} 
                    onChange={handleChange} 
                    required
                    className="w-full px-3 py-2 text-sm border border-slate-300 rounded-lg focus:ring-2 focus:ring-blue-500 outline-none" 
                    placeholder="예: 강서K병원" 
                  />
                </div>

                {/* 9. 가입번호 (고객명 바로 우측 배치) */}
                <div className="space-y-1">
                  <label className="text-xs font-semibold text-blue-700 flex items-center gap-1">
                    <Hash className="w-3.5 h-3.5 text-blue-600" /> 가입번호 (마스킹 대상)
                  </label>
                  <input 
                    type="text" 
                    name="subNumber" 
                    value={formData.subNumber} 
                    onChange={handleChange} 
                    required
                    className="w-full px-3 py-2 text-sm font-mono border border-blue-300 bg-blue-50/30 rounded-lg focus:ring-2 focus:ring-blue-500 focus:border-blue-500 outline-none" 
                    placeholder="예: 500-2404-1001" 
                  />
                </div>

                {/* 10. 주소 (Full Width) */}
                <div className="space-y-1 md:col-span-3">
                  <label className="text-xs font-semibold text-slate-700 flex items-center gap-1">
                    <MapPin className="w-3.5 h-3.5 text-slate-400" /> 주소 (마스킹 대상)
                  </label>
                  <input 
                    type="text" 
                    name="address" 
                    value={formData.address} 
                    onChange={handleChange} 
                    required
                    className="w-full px-3 py-2 text-sm border border-slate-300 rounded-lg focus:ring-2 focus:ring-blue-500 outline-none" 
                    placeholder="상세 주소를 입력하세요" 
                  />
                </div>

                {/* 11. 회선수 */}
                <div className="space-y-1">
                  <label className="text-xs font-semibold text-slate-700 flex items-center gap-1">
                    <Check className="w-3.5 h-3.5 text-slate-400" /> 회선수
                  </label>
                  <input 
                    type="number" 
                    name="lines" 
                    value={formData.lines} 
                    onChange={handleChange} 
                    required 
                    min="1"
                    className="w-full px-3 py-2 text-sm border border-slate-300 rounded-lg focus:ring-2 focus:ring-blue-500 outline-none" 
                    placeholder="숫자 입력" 
                  />
                </div>

                {/* 12. 기본 단가 (35,000원 - 회선수 우측 배치) */}
                <div className="space-y-1">
                  <label className="text-xs font-semibold text-slate-700 flex items-center justify-between">
                    <span>기본 단가 (원)</span>
                    <span className="text-[11px] font-bold text-blue-600">
                      적용단가: {((Number(formData.unitPrice) || BASE_UNIT_PRICE) + (formData.isDifficult ? DIFFICULT_EXTRA_PRICE : 0)).toLocaleString()}원
                    </span>
                  </label>
                  <input
                    type="number"
                    name="unitPrice"
                    value={formData.unitPrice}
                    onChange={handleChange}
                    required
                    step="1000"
                    className="w-full px-3 py-2 text-sm font-mono font-bold border border-slate-300 rounded-lg focus:ring-2 focus:ring-blue-500 outline-none"
                    placeholder="35000"
                  />
                </div>

                {/* 13. 난개통 체크박스 (+10,000원) */}
                <div className="flex items-end">
                  <label className={`flex items-center justify-between gap-2 cursor-pointer select-none px-3.5 py-2 rounded-lg border w-full transition-colors ${
                    formData.isDifficult
                      ? 'bg-orange-100 border-orange-400 text-orange-950'
                      : 'bg-orange-50/50 border-orange-200 text-slate-700'
                  }`}>
                    <div className="flex items-center gap-2">
                      <input
                        type="checkbox"
                        name="isDifficult"
                        checked={Boolean(formData.isDifficult)}
                        onChange={handleChange}
                        className="w-4 h-4 text-orange-600 rounded focus:ring-orange-500"
                      />
                      <span className="text-xs font-extrabold">난개통 추가단가</span>
                    </div>
                    <span className="text-xs font-mono font-bold text-orange-700">+10,000원</span>
                  </label>
                </div>

                {/* 14. 예상 정산금액 실시간 미리보기 배너 */}
                <div className="md:col-span-3 bg-slate-50 border border-slate-200 rounded-lg px-4 py-2.5 flex items-center justify-between text-xs sm:text-sm">
                  <span className="font-semibold text-slate-600">
                    💰 산출 공식: 회선수({Number(formData.lines) || 0}회선) × 적용단가({((Number(formData.unitPrice) || BASE_UNIT_PRICE) + (formData.isDifficult ? DIFFICULT_EXTRA_PRICE : 0)).toLocaleString()}원)
                  </span>
                  <span className="font-mono font-extrabold text-blue-700 text-sm sm:text-base">
                    총 정산금액: {((Number(formData.lines) || 0) * ((Number(formData.unitPrice) || BASE_UNIT_PRICE) + (formData.isDifficult ? DIFFICULT_EXTRA_PRICE : 0))).toLocaleString()}원
                  </span>
                </div>

                {/* 15. 개통예정일 */}
                <div className="space-y-1">
                  <label className="text-xs font-semibold text-slate-700 flex items-center gap-1">
                    <Calendar className="w-3.5 h-3.5 text-slate-400" /> 개통예정일
                  </label>
                  <input 
                    type="text" 
                    name="scheduledDate" 
                    value={formData.scheduledDate} 
                    onChange={handleChange} 
                    required
                    className="w-full px-3 py-2 text-sm border border-slate-300 rounded-lg focus:ring-2 focus:ring-blue-500 outline-none" 
                    placeholder="YYYY-MM-DD 또는 미정" 
                  />
                </div>

                {/* 16. 비고 (정산상태 선택) */}
                <div className="space-y-1">
                  <label className="text-xs font-semibold text-slate-700 flex items-center gap-1">
                    <FileText className="w-3.5 h-3.5 text-slate-400" /> 비고 (정산상태)
                  </label>
                  <select
                    name="remarks"
                    value={formData.remarks}
                    onChange={handleChange}
                    className="w-full px-3 py-2 text-sm font-bold text-slate-800 border border-slate-300 bg-slate-50/50 rounded-lg focus:ring-2 focus:ring-blue-500 outline-none cursor-pointer"
                  >
                    {!REMARKS_STATUS_LIST.includes(formData.remarks) && formData.remarks && (
                      <option value={formData.remarks}>{formData.remarks}</option>
                    )}
                    {REMARKS_STATUS_LIST.map((statusOption) => (
                      <option key={statusOption} value={statusOption}>
                        {statusOption}
                      </option>
                    ))}
                  </select>
                </div>

                {/* 17. 추가 메모 & 형광펜 강조 */}
                <div className="space-y-1 md:col-span-2">
                  <label className="text-xs font-semibold text-slate-700 flex items-center gap-1">
                    우측 추가메모 (예: 12월)
                  </label>
                  <input 
                    type="text" 
                    name="extraNote" 
                    value={formData.extraNote} 
                    onChange={handleChange}
                    className="w-full px-3 py-2 text-sm border border-slate-300 rounded-lg focus:ring-2 focus:ring-blue-500 outline-none" 
                    placeholder="예: 12월 (선택사항)" 
                  />
                </div>

                <div className="flex items-end pb-1">
                  <label className="flex items-center gap-2 cursor-pointer select-none px-3 py-2 rounded-lg bg-yellow-50 border border-yellow-300 w-full">
                    <input 
                      type="checkbox" 
                      name="isHighlighted" 
                      checked={formData.isHighlighted} 
                      onChange={handleChange}
                      className="w-4 h-4 text-blue-600 rounded focus:ring-blue-500"
                    />
                    <span className="text-xs font-bold text-slate-800">노란색 형광펜 강조</span>
                  </label>
                </div>

                {/* 18. 사진 / 엑셀 / PDF 파일 첨부 영역 */}
                <div className="md:col-span-3 pt-2 border-t border-slate-200">
                  <div className="flex flex-wrap items-center justify-between gap-2 mb-2">
                    <label className="text-xs font-bold text-slate-700 flex items-center gap-1.5">
                      <Paperclip className="w-3.5 h-3.5 text-blue-600" />
                      첨부파일 등록 (사진 · 엑셀 · PDF)
                    </label>
                    <label className="cursor-pointer inline-flex items-center gap-1.5 px-3 py-1.5 rounded-lg text-xs font-bold bg-blue-50 text-blue-700 border border-blue-200 hover:bg-blue-100 transition-colors">
                      <Upload className="w-3.5 h-3.5" />
                      <span>{uploadingFile ? '처리 중...' : '+ 파일 선택 (사진/엑셀/PDF)'}</span>
                      <input
                        type="file"
                        multiple
                        accept="image/*,.xlsx,.xls,.csv,.pdf,application/pdf,application/vnd.ms-excel,application/vnd.openxmlformats-officedocument.spreadsheetml.sheet"
                        onChange={handleFormAttachmentUpload}
                        className="hidden"
                      />
                    </label>
                  </div>

                  {(!formData.attachments || formData.attachments.length === 0) ? (
                    <div className="text-xs text-slate-400 bg-slate-50 border border-dashed border-slate-300 rounded-lg py-3 px-4 text-center">
                      첨부된 파일이 없습니다. 현장 사진(JPG/PNG), 엑셀 정산서(XLSX/CSV), 계약서(PDF)를 첨부할 수 있습니다.
                    </div>
                  ) : (
                    <div className="grid grid-cols-1 sm:grid-cols-2 gap-2">
                      {formData.attachments.map((att) => (
                        <div key={att.id} className="flex items-center justify-between gap-2 px-3 py-2 rounded-lg border border-slate-200 bg-slate-50 text-xs">
                          <div className="flex items-center gap-2 min-w-0">
                            {att.category === 'IMAGE' && <Image className="w-4 h-4 text-blue-600 shrink-0" />}
                            {att.category === 'EXCEL' && <FileSpreadsheet className="w-4 h-4 text-emerald-600 shrink-0" />}
                            {att.category === 'PDF' && <FileText className="w-4 h-4 text-red-600 shrink-0" />}
                            <span className="font-semibold text-slate-800 truncate" title={att.name}>{att.name}</span>
                            <span className="text-[10px] text-slate-400 shrink-0">({formatFileSize(att.size)})</span>
                          </div>
                          <button
                            type="button"
                            onClick={() => handleRemoveFormAttachment(att.id)}
                            className="text-slate-400 hover:text-red-600 p-1"
                            title="첨부 취소"
                          >
                            <X className="w-3.5 h-3.5" />
                          </button>
                        </div>
                      ))}
                    </div>
                  )}
                </div>
              </div>

              <div className="mt-6 pt-4 border-t border-slate-200 flex justify-end gap-2">
                <button 
                  type="button" 
                  onClick={handleCloseModal}
                  className="px-4 py-2 text-sm rounded-lg border border-slate-300 text-slate-700 font-medium hover:bg-slate-50 transition-colors"
                >
                  취소
                </button>
                <button 
                  type="submit"
                  className="px-5 py-2 text-sm rounded-lg bg-blue-600 text-white font-semibold hover:bg-blue-700 transition-colors shadow-sm"
                >
                  {editingId ? '수정 내용 저장' : '내역 등록 완료'}
                </button>
              </div>
            </form>
          </div>
        </div>
      )}

      {/* 관리란 전용 첨부파일 관리 모달 (사진 / 엑셀 / PDF 업로드, 미리보기, 다운로드) */}
      {activeAttachmentRecord && (
        <div className="fixed inset-0 z-50 flex items-center justify-center p-4 bg-slate-900/60 backdrop-blur-sm overflow-y-auto">
          <div className="bg-white rounded-2xl shadow-2xl w-full max-w-3xl overflow-hidden my-8 border border-slate-200">
            {/* Modal Header */}
            <div className="px-6 py-4 border-b border-slate-200 bg-slate-900 text-white flex items-center justify-between">
              <div className="flex items-center gap-2.5">
                <div className="p-2 rounded-lg bg-blue-500/20 border border-blue-400/30 text-blue-300">
                  <Paperclip className="w-5 h-5" />
                </div>
                <div>
                  <h3 className="text-base sm:text-lg font-bold flex items-center gap-2">
                    <span>[{activeAttachmentRecord.customer}] 첨부자료 관리</span>
                    <span className="text-xs px-2 py-0.5 rounded bg-slate-800 text-blue-300 font-normal">
                      담당: {activeAttachmentRecord.staffName || '-'}
                    </span>
                  </h3>
                  <p className="text-xs text-slate-400">
                    현장 시공 사진(JPG·PNG), 엑셀 내역서(XLSX·XLS·CSV), 관련 문서(PDF)를 첨부하고 열람할 수 있습니다.
                  </p>
                </div>
              </div>
              <button
                onClick={() => {
                  setAttachmentModalRecordId(null);
                  setPreviewAttachment(null);
                  setAttachmentNotice({ type: '', message: '' });
                }}
                className="text-slate-400 hover:text-white p-1.5 rounded-full hover:bg-slate-800 transition-colors"
              >
                <X className="w-5 h-5" />
              </button>
            </div>

            <div className="p-6 space-y-5 max-h-[78vh] overflow-y-auto">
              {/* Notice Alert */}
              {attachmentNotice.message && (
                <div className={`p-3.5 rounded-xl border text-xs sm:text-sm font-medium flex items-center justify-between gap-2 ${
                  attachmentNotice.type === 'error'
                    ? 'bg-red-50 border-red-200 text-red-700'
                    : 'bg-emerald-50 border-emerald-200 text-emerald-800'
                }`}>
                  <div className="flex items-center gap-2">
                    <AlertCircle className="w-4 h-4 shrink-0" />
                    <span>{attachmentNotice.message}</span>
                  </div>
                  <button onClick={() => setAttachmentNotice({ type: '', message: '' })} className="text-slate-400 hover:text-slate-700">
                    <X className="w-4 h-4" />
                  </button>
                </div>
              )}

              {/* Quick Upload Buttons by File Type (사진 / 엑셀 / PDF) */}
              <div className="grid grid-cols-1 sm:grid-cols-3 gap-3">
                {/* 1. 사진 첨부 */}
                <label className="cursor-pointer flex flex-col items-center justify-center p-4 rounded-xl border-2 border-dashed border-blue-300 bg-blue-50/40 hover:bg-blue-50 transition-colors text-center">
                  <Image className="w-7 h-7 text-blue-600 mb-1.5" />
                  <span className="text-xs sm:text-sm font-extrabold text-blue-900">사진 파일 첨부</span>
                  <span className="text-[11px] text-slate-500 mt-0.5">JPG, PNG, WEBP (자동 압축)</span>
                  <input
                    type="file"
                    accept="image/*"
                    multiple
                    disabled={uploadingFile}
                    onChange={(e) => handleDirectRecordAttachmentUpload(e, activeAttachmentRecord)}
                    className="hidden"
                  />
                </label>

                {/* 2. 엑셀 파일 첨부 */}
                <label className="cursor-pointer flex flex-col items-center justify-center p-4 rounded-xl border-2 border-dashed border-emerald-300 bg-emerald-50/40 hover:bg-emerald-50 transition-colors text-center">
                  <FileSpreadsheet className="w-7 h-7 text-emerald-600 mb-1.5" />
                  <span className="text-xs sm:text-sm font-extrabold text-emerald-900">엑셀 파일 첨부</span>
                  <span className="text-[11px] text-slate-500 mt-0.5">XLSX, XLS, CSV 정산서</span>
                  <input
                    type="file"
                    accept=".xlsx,.xls,.csv,application/vnd.ms-excel,application/vnd.openxmlformats-officedocument.spreadsheetml.sheet,text/csv"
                    multiple
                    disabled={uploadingFile}
                    onChange={(e) => handleDirectRecordAttachmentUpload(e, activeAttachmentRecord)}
                    className="hidden"
                  />
                </label>

                {/* 3. PDF 파일 첨부 */}
                <label className="cursor-pointer flex flex-col items-center justify-center p-4 rounded-xl border-2 border-dashed border-red-300 bg-red-50/40 hover:bg-red-50 transition-colors text-center">
                  <FileText className="w-7 h-7 text-red-600 mb-1.5" />
                  <span className="text-xs sm:text-sm font-extrabold text-red-900">PDF 문서 첨부</span>
                  <span className="text-[11px] text-slate-500 mt-0.5">개통서류 · 계약서 (.pdf)</span>
                  <input
                    type="file"
                    accept=".pdf,application/pdf"
                    multiple
                    disabled={uploadingFile}
                    onChange={(e) => handleDirectRecordAttachmentUpload(e, activeAttachmentRecord)}
                    className="hidden"
                  />
                </label>
              </div>

              {/* Attached File List */}
              <div>
                <div className="flex items-center justify-between mb-2.5">
                  <h4 className="text-xs sm:text-sm font-extrabold text-slate-800">
                    등록된 첨부파일 목록 ({(activeAttachmentRecord.attachments || []).length}건)
                  </h4>
                  {uploadingFile && (
                    <span className="text-xs font-bold text-blue-600 animate-pulse">파일 업로드 및 변환 중...</span>
                  )}
                </div>

                {(!activeAttachmentRecord.attachments || activeAttachmentRecord.attachments.length === 0) ? (
                  <div className="py-10 bg-slate-50 rounded-xl border border-slate-200 text-center text-slate-400 text-xs sm:text-sm">
                    등록된 첨부파일이 없습니다. 상단 버튼을 클릭해 사진, 엑셀, PDF 파일을 추가해보세요.
                  </div>
                ) : (
                  <div className="space-y-2.5">
                    {activeAttachmentRecord.attachments.map((att) => (
                      <div
                        key={att.id}
                        className="flex flex-col sm:flex-row sm:items-center justify-between gap-3 p-3 rounded-xl border border-slate-200 bg-white hover:border-blue-300 transition-colors shadow-sm"
                      >
                        <div className="flex items-center gap-3 min-w-0">
                          {/* Thumbnail or File Icon */}
                          {att.category === 'IMAGE' ? (
                            <img
                              src={att.dataUrl}
                              alt={att.name}
                              onClick={() => setPreviewAttachment(att)}
                              className="w-12 h-12 rounded-lg object-cover border border-slate-200 cursor-pointer shrink-0 hover:opacity-80"
                            />
                          ) : att.category === 'EXCEL' ? (
                            <div className="w-12 h-12 rounded-lg bg-emerald-100 text-emerald-700 flex items-center justify-center shrink-0 font-bold">
                              <FileSpreadsheet className="w-6 h-6" />
                            </div>
                          ) : (
                            <div className="w-12 h-12 rounded-lg bg-red-100 text-red-700 flex items-center justify-center shrink-0 font-bold">
                              <FileText className="w-6 h-6" />
                            </div>
                          )}

                          <div className="min-w-0">
                            <div className="flex items-center gap-1.5">
                              <span className={`px-1.5 py-0.5 rounded text-[10px] font-extrabold ${
                                att.category === 'IMAGE'
                                  ? 'bg-blue-100 text-blue-800'
                                  : att.category === 'EXCEL'
                                  ? 'bg-emerald-100 text-emerald-800'
                                  : 'bg-red-100 text-red-800'
                              }`}>
                                {att.category === 'IMAGE' ? '사진' : att.category === 'EXCEL' ? '엑셀' : 'PDF'}
                              </span>
                              <p className="text-xs sm:text-sm font-bold text-slate-900 truncate" title={att.name}>
                                {att.name}
                              </p>
                            </div>
                            <p className="text-[11px] text-slate-500 mt-0.5">
                              용량: {formatFileSize(att.size)} · 등록일: {att.uploadedAt || '-'} · 등록자: {att.uploadedBy || '-'}
                            </p>
                          </div>
                        </div>

                        <div className="flex items-center justify-end gap-1.5 shrink-0">
                          {(att.category === 'IMAGE' || att.category === 'PDF') && (
                            <button
                              type="button"
                              onClick={() => setPreviewAttachment(previewAttachment?.id === att.id ? null : att)}
                              className="px-2.5 py-1.5 rounded-lg text-xs font-bold bg-slate-100 text-slate-700 hover:bg-slate-200 flex items-center gap-1 transition-colors"
                            >
                              <Eye className="w-3.5 h-3.5" />
                              <span>{previewAttachment?.id === att.id ? '미리보기 닫기' : '미리보기'}</span>
                            </button>
                          )}
                          <button
                            type="button"
                            onClick={() => handleDownloadAttachment(att)}
                            className="px-2.5 py-1.5 rounded-lg text-xs font-bold bg-blue-600 text-white hover:bg-blue-700 flex items-center gap-1 transition-colors"
                          >
                            <Download className="w-3.5 h-3.5" />
                            <span>다운로드</span>
                          </button>
                          <button
                            type="button"
                            onClick={() => handleDeleteRecordAttachment(activeAttachmentRecord, att.id)}
                            className="p-1.5 rounded-lg text-slate-400 hover:text-red-600 hover:bg-red-50 transition-colors"
                            title="파일 삭제"
                          >
                            <Trash2 className="w-4 h-4" />
                          </button>
                        </div>
                      </div>
                    ))}
                  </div>
                )}
              </div>

              {/* Preview Box for Image or PDF */}
              {previewAttachment && (
                <div className="p-4 rounded-xl bg-slate-900 text-white space-y-3">
                  <div className="flex items-center justify-between">
                    <span className="text-xs font-bold text-blue-300 flex items-center gap-1.5">
                      <Eye className="w-4 h-4" /> 미리보기: {previewAttachment.name}
                    </span>
                    <button
                      type="button"
                      onClick={() => setPreviewAttachment(null)}
                      className="text-xs text-slate-400 hover:text-white"
                    >
                      닫기 ✕
                    </button>
                  </div>
                  {previewAttachment.category === 'IMAGE' ? (
                    <div className="flex justify-center bg-slate-950 rounded-lg p-2">
                      <img
                        src={previewAttachment.dataUrl}
                        alt={previewAttachment.name}
                        className="max-h-96 object-contain rounded"
                      />
                    </div>
                  ) : previewAttachment.category === 'PDF' ? (
                    <iframe
                      title={previewAttachment.name}
                      src={previewAttachment.dataUrl}
                      className="w-full h-96 rounded-lg bg-white"
                    />
                  ) : null}
                </div>
              )}
            </div>

            <div className="px-6 py-3 bg-slate-50 border-t border-slate-200 flex justify-between items-center text-xs text-slate-500">
              <span>💡 첨부된 파일은 클라우드에 자동 저장되며 언제든 다운로드할 수 있습니다.</span>
              <button
                type="button"
                onClick={() => {
                  setAttachmentModalRecordId(null);
                  setPreviewAttachment(null);
                }}
                className="px-4 py-1.5 bg-slate-800 text-white font-semibold rounded-lg hover:bg-slate-700"
              >
                닫기
              </button>
            </div>
          </div>
        </div>
      )}

      {/* Delete Confirmation Modal */}
      {deleteConfirmId && (
        <div className="fixed inset-0 z-50 flex items-center justify-center p-4 bg-slate-900/50 backdrop-blur-sm">
          <div className="bg-white rounded-xl shadow-xl w-full max-w-sm p-6 text-center">
            <div className="w-12 h-12 rounded-full bg-red-100 text-red-600 flex items-center justify-center mx-auto mb-4">
              <AlertCircle className="w-6 h-6" />
            </div>
            <h3 className="text-lg font-bold text-slate-900 mb-2">해당 내역을 삭제하시겠습니까?</h3>
            <p className="text-slate-500 text-sm mb-6">삭제된 데이터는 복구할 수 없습니다.</p>
            <div className="flex gap-3 justify-center">
              <button 
                onClick={() => setDeleteConfirmId(null)}
                className="flex-1 px-4 py-2 border border-slate-300 rounded-lg text-slate-700 font-medium hover:bg-slate-50 transition-colors"
              >
                취소
              </button>
              <button 
                onClick={() => handleDelete(deleteConfirmId)}
                className="flex-1 px-4 py-2 bg-red-600 rounded-lg text-white font-medium hover:bg-red-700 transition-colors"
              >
                삭제하기
              </button>
            </div>
          </div>
        </div>
      )}

      {/* Reset to Original 27 Records Modal */}
      {resetConfirmOpen && (
        <div className="fixed inset-0 z-50 flex items-center justify-center p-4 bg-slate-900/50 backdrop-blur-sm">
          <div className="bg-white rounded-xl shadow-xl w-full max-w-sm p-6 text-center">
            <div className="w-12 h-12 rounded-full bg-blue-100 text-blue-600 flex items-center justify-center mx-auto mb-4">
              <RefreshCw className="w-6 h-6" />
            </div>
            <h3 className="text-lg font-bold text-slate-900 mb-2">원본 데이터(27건)로 복원할까요?</h3>
            <p className="text-slate-500 text-xs sm:text-sm mb-6">
              엑셀 파일의 27개 구내공사 내역과 가입번호 기본값으로 전체 리스트를 초기화합니다.
            </p>
            <div className="flex gap-3 justify-center">
              <button 
                onClick={() => setResetConfirmOpen(false)}
                className="flex-1 px-4 py-2 border border-slate-300 rounded-lg text-slate-700 font-medium hover:bg-slate-50 transition-colors"
              >
                취소
              </button>
              <button 
                onClick={handleResetToExcelData}
                className="flex-1 px-4 py-2 bg-blue-600 rounded-lg text-white font-medium hover:bg-blue-700 transition-colors"
              >
                복원 실행
              </button>
            </div>
          </div>
        </div>
      )}

      {/* Password Modal for Unmasking */}
      {showPasswordModal && (
        <div className="fixed inset-0 z-50 flex items-center justify-center p-4 bg-slate-900/60 backdrop-blur-sm">
          <div className="bg-white rounded-xl shadow-xl w-full max-w-sm overflow-hidden border border-slate-200">
            <div className="px-6 py-4 border-b border-slate-200 bg-slate-900 text-white flex justify-between items-center">
              <h3 className="text-base font-bold flex items-center gap-2">
                <Key className="w-5 h-5 text-amber-400" />
                보안 인증 (마스킹 해제)
              </h3>
              <button onClick={() => setShowPasswordModal(false)} className="text-slate-400 hover:text-white">
                <X className="w-5 h-5" />
              </button>
            </div>
            <form onSubmit={handlePasswordSubmit} className="p-6">
              <p className="text-xs sm:text-sm text-slate-600 mb-4 leading-relaxed">
                <strong>가입번호, 연락처, 상세주소</strong>의 마스킹을 해제하려면 관리자 비밀번호를 입력해주세요.
              </p>
              <input
                type="password"
                autoFocus
                placeholder="관리자 비밀번호를 입력하세요"
                value={passwordInput}
                onChange={(e) => {
                  setPasswordInput(e.target.value);
                  setPasswordError('');
                }}
                className={`w-full px-4 py-2.5 border rounded-lg focus:outline-none focus:ring-2 mb-2 font-mono text-center tracking-widest ${
                  passwordError ? 'border-red-500 focus:ring-red-200' : 'border-slate-300 focus:ring-blue-200 focus:border-blue-500'
                }`}
              />
              {passwordError && (
                <p className="text-red-500 text-xs font-medium mb-3 text-center">{passwordError}</p>
              )}
              <div className="mt-5 flex justify-end gap-2">
                <button
                  type="button"
                  onClick={() => setShowPasswordModal(false)}
                  className="px-4 py-2 text-sm rounded-lg border border-slate-300 text-slate-700 hover:bg-slate-50 transition-colors"
                >
                  취소
                </button>
                <button
                  type="submit"
                  className="px-5 py-2 text-sm rounded-lg bg-blue-600 text-white font-semibold hover:bg-blue-700 transition-colors"
                >
                  잠금 해제
                </button>
              </div>
            </form>
          </div>
        </div>
      )}

      {/* 직원 공유 및 동시 접속 가이드 모달 */}
      {isShareModalOpen && (
        <div className="fixed inset-0 z-50 flex items-center justify-center p-4 bg-slate-900/60 backdrop-blur-sm overflow-y-auto">
          <div className="bg-white rounded-2xl shadow-2xl w-full max-w-2xl overflow-hidden my-8 border border-slate-200">
            <div className="px-6 py-4 border-b border-slate-200 bg-slate-900 text-white flex items-center justify-between">
              <div className="flex items-center gap-2.5">
                <div className="p-2 rounded-lg bg-emerald-500/20 border border-emerald-400/30 text-emerald-300">
                  <Share2 className="w-5 h-5" />
                </div>
                <div>
                  <h3 className="text-base sm:text-lg font-bold">직원 공유 및 다중 동시접속 안내</h3>
                  <p className="text-xs text-slate-400">전 직원이 스마트폰·PC로 실시간 공유하는 방법입니다.</p>
                </div>
              </div>
              <button
                onClick={() => setIsShareModalOpen(false)}
                className="text-slate-400 hover:text-white p-1 rounded-full hover:bg-slate-800"
              >
                <X className="w-5 h-5" />
              </button>
            </div>

            <div className="p-6 space-y-5 text-xs sm:text-sm text-slate-700">
              {/* 1. 실시간 공용 DB 적용 완료 안내 */}
              <div className="p-4 rounded-xl bg-emerald-50 border border-emerald-200 flex items-start gap-3">
                <CheckCircle2 className="w-5 h-5 text-emerald-600 shrink-0 mt-0.5" />
                <div>
                  <p className="font-extrabold text-emerald-950">
                    전사 공용 클라우드 DB 실시간 연동이 활성화되었습니다!
                  </p>
                  <p className="text-xs text-emerald-800 mt-1 leading-relaxed">
                    이제 대표님과 직원분들이 각자의 휴대폰이나 PC에서 접속해도 <strong>동일한 구내공사 리스트, 회원승인 내역, 첨부파일(사진·엑셀·PDF)</strong>을 실시간으로 함께 보고 수정할 수 있습니다.
                  </p>
                </div>
              </div>

              {/* 2. 단계별 공유 방법 */}
              <div className="space-y-3">
                <h4 className="font-extrabold text-slate-900 flex items-center gap-1.5">
                  <Globe className="w-4 h-4 text-blue-600" />
                  직원분들과 프로그램을 공유하는 2가지 방법
                </h4>

                <div className="grid grid-cols-1 md:grid-cols-2 gap-3">
                  <div className="p-4 rounded-xl border border-slate-200 bg-slate-50">
                    <div className="font-bold text-blue-700 mb-1 flex items-center gap-1.5">
                      <Share2 className="w-4 h-4" /> 방법 1. 현재 화면의 공유 버튼 활용
                    </div>
                    <p className="text-xs text-slate-600 leading-relaxed">
                      우측 상단 에디터 메뉴의 <strong>[공유(Share)]</strong> 또는 <strong>[공개 링크 생성]</strong> 버튼을 눌러 생성된 주소를 카카오톡 단톡방에 전달하면 직원들이 바로 접속할 수 있습니다.
                    </p>
                  </div>

                  <div className="p-4 rounded-xl border border-slate-200 bg-slate-50">
                    <div className="font-bold text-indigo-700 mb-1 flex items-center gap-1.5">
                      <Smartphone className="w-4 h-4" /> 방법 2. 무료 웹 호스팅(Vercel 등) 배포
                    </div>
                    <p className="text-xs text-slate-600 leading-relaxed">
                      고유 도메인 주소(예: <code className="bg-slate-200 px-1 rounded">ent-iptv.vercel.app</code>)로 정식 배포하여 스마트폰 홈 화면에 앱처럼 추가해 영구적으로 사용할 수도 있습니다.
                    </p>
                  </div>
                </div>
              </div>

              {/* 3. 직원별 접속 및 권한 프로세스 */}
              <div className="p-4 rounded-xl border border-slate-200 bg-white space-y-2">
                <div className="flex items-center justify-between">
                  <span className="font-extrabold text-slate-900">📋 직원 단톡방 공지용 안내문구</span>
                  <button
                    type="button"
                    onClick={handleCopyShareNotice}
                    className="inline-flex items-center gap-1.5 px-3 py-1.5 rounded-lg text-xs font-bold bg-blue-600 hover:bg-blue-700 text-white transition-colors"
                  >
                    <Copy className="w-3.5 h-3.5" />
                    <span>{copySuccess ? '복사 완료!' : '공지문구 복사하기'}</span>
                  </button>
                </div>
                <div className="p-3 rounded-lg bg-slate-100 text-xs text-slate-700 font-mono whitespace-pre-line leading-relaxed">
                  {`[📢 (주)이앤티 단체형IPTV 구내공사 관리시스템 접속 안내]
1. 공유된 링크를 통해 스마트폰 또는 PC로 접속합니다.
2. 처음 접속하시는 직원은 [회원가입 신청]에서 본인 실명(예: 박국용, 박정훈 등)으로 신청해주세요.
3. 대표님 승인 후 로그인하면 본인 담당 공사 내역 조회·수정 및 현장 사진/엑셀/PDF 첨부가 실시간 연동됩니다.`}
                </div>
              </div>
            </div>

            <div className="px-6 py-3 bg-slate-50 border-t border-slate-200 flex justify-between items-center text-xs text-slate-500">
              <span>🔒 일반 직원은 로그인 시 본인 이름의 공사 내역만 열람·수정할 수 있어 안전합니다.</span>
              <button
                type="button"
                onClick={() => setIsShareModalOpen(false)}
                className="px-4 py-1.5 bg-slate-800 text-white font-semibold rounded-lg hover:bg-slate-700"
              >
                확인
              </button>
            </div>
          </div>
        </div>
      )}
    </div>
  );
}
