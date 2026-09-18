namespace App\Http\Controllers;

use App\Models\XeMay;
use Illuminate\Http\Request;
use Illuminate\Support\Facades\Validator;

class XeMayController extends Controller
{
    /**
     * Hiển thị danh sách tất cả xe máy
     */
    public function index()
    {
        $xeMay = XeMay::orderBy('id', 'desc')->get();

        return response()->json([
            'success' => true,
            'message' => 'Lấy danh sách xe máy thành công',
            'total' => $xeMay->count(),
            'data' => $xeMay
        ], 200);
    }


    /**
     * Hiển thị thông tin chi tiết của một xe
     */
    public function show($id)
    {
        $xe = XeMay::find($id);

        if (!$xe) {
            return response()->json([
                'success' => false,
                'message' => 'Không tìm thấy xe máy có mã: ' . $id
            ], 404);
        }

        return response()->json([
            'success' => true,
            'message' => 'Lấy thông tin xe thành công',
            'data' => $xe
        ], 200);
    }


    /**
     * Thêm một xe máy mới
     */
    public function store(Request $request)
    {
        $validator = Validator::make($request->all(), [
            'ten_xe' => 'required|string|max:255',
            'bien_so' => 'required|string|max:20|unique:xe_mays,bien_so',
            'gia_thue' => 'required|numeric|min:0',
            'trang_thai' => 'required|string',
            'mo_ta' => 'nullable|string',
            'hinh_anh' => 'nullable|string',
            'danh_muc_id' => 'required|integer'
        ], [
            'ten_xe.required' => 'Vui lòng nhập tên xe',
            'bien_so.required' => 'Vui lòng nhập biển số xe',
            'bien_so.unique' => 'Biển số xe đã tồn tại',
            'gia_thue.required' => 'Vui lòng nhập giá thuê',
            'gia_thue.numeric' => 'Giá thuê phải là số',
            'gia_thue.min' => 'Giá thuê không được nhỏ hơn 0',
            'trang_thai.required' => 'Vui lòng nhập trạng thái xe',
            'danh_muc_id.required' => 'Vui lòng chọn danh mục xe'
        ]);

        if ($validator->fails()) {
            return response()->json([
                'success' => false,
                'message' => 'Dữ liệu không hợp lệ',
                'errors' => $validator->errors()
            ], 422);
        }

        $xe = new XeMay();

        $xe->ten_xe = $request->ten_xe;
        $xe->bien_so = $request->bien_so;
        $xe->gia_thue = $request->gia_thue;
        $xe->trang_thai = $request->trang_thai;
        $xe->mo_ta = $request->mo_ta;
        $xe->hinh_anh = $request->hinh_anh;
        $xe->danh_muc_id = $request->danh_muc_id;

        $xe->save();

        return response()->json([
            'success' => true,
            'message' => 'Thêm xe máy thành công',
            'data' => $xe
        ], 201);
    }


    /**
     * Cập nhật thông tin xe máy
     */
    public function update(Request $request, $id)
    {
        $xe = XeMay::find($id);

        if (!$xe) {
            return response()->json([
                'success' => false,
                'message' => 'Không tìm thấy xe cần cập nhật'
            ], 404);
        }

        $validator = Validator::make($request->all(), [
            'ten_xe' => 'required|string|max:255',
            'bien_so' => 'required|string|max:20',
            'gia_thue' => 'required|numeric|min:0',
            'trang_thai' => 'required|string',
            'mo_ta' => 'nullable|string',
            'hinh_anh' => 'nullable|string',
            'danh_muc_id' => 'required|integer'
        ]);

        if ($validator->fails()) {
            return response()->json([
                'success' => false,
                'message' => 'Dữ liệu cập nhật không hợp lệ',
                'errors' => $validator->errors()
            ], 422);
        }

        $xe->ten_xe = $request->ten_xe;
        $xe->bien_so = $request->bien_so;
        $xe->gia_thue = $request->gia_thue;
        $xe->trang_thai = $request->trang_thai;
        $xe->mo_ta = $request->mo_ta;
        $xe->hinh_anh = $request->hinh_anh;
        $xe->danh_muc_id = $request->danh_muc_id;

        $xe->save();

        return response()->json([
            'success' => true,
            'message' => 'Cập nhật xe máy thành công',
            'data' => $xe
        ], 200);
    }


    /**
     * Xóa xe máy
     */
    public function destroy($id)
    {
        $xe = XeMay::find($id);

        if (!$xe) {
            return response()->json([
                'success' => false,
                'message' => 'Không tìm thấy xe cần xóa'
            ], 404);
        }

        $xe->delete();

        return response()->json([
            'success' => true,
            'message' => 'Xóa xe máy thành công'
        ], 200);
    }


    /**
     * Tìm kiếm xe máy
     */
    public function search(Request $request)
    {
        $keyword = $request->keyword;

        if (!$keyword) {
            return response()->json([
                'success' => false,
                'message' => 'Vui lòng nhập từ khóa tìm kiếm'
            ], 400);
        }

        $xeMay = XeMay::where('ten_xe', 'LIKE', '%' . $keyword . '%')
            ->orWhere('bien_so', 'LIKE', '%' . $keyword . '%')
            ->get();

        return response()->json([
            'success' => true,
            'message' => 'Kết quả tìm kiếm',
            'keyword' => $keyword,
            'total' => $xeMay->count(),
            'data' => $xeMay
        ], 200);
    }


    /**
     * Lọc xe theo trạng thái
     */
    public function filterByStatus($status)
    {
        $xeMay = XeMay::where('trang_thai', $status)
            ->orderBy('ten_xe', 'asc')
            ->get();

        return response()->json([
            'success' => true,
            'message' => 'Lọc xe theo trạng thái thành công',
            'status' => $status,
            'total' => $xeMay->count(),
            'data' => $xeMay
        ], 200);
    }


    /**
     * Lấy danh sách xe đang sẵn sàng cho thuê
     */
    public function available()
    {
        $xeMay = XeMay::where('trang_thai', 'Sẵn sàng')
            ->orderBy('ten_xe', 'asc')
            ->get();

        return response()->json([
            'success' => true,
            'message' => 'Danh sách xe đang sẵn sàng',
            'total' => $xeMay->count(),
            'data' => $xeMay
        ], 200);
    }


    /**
     * Thay đổi trạng thái xe
     */
    public function changeStatus(Request $request, $id)
    {
        $xe = XeMay::find($id);

        if (!$xe) {
            return response()->json([
                'success' => false,
                'message' => 'Không tìm thấy xe máy'
            ], 404);
        }

        $validator = Validator::make($request->all(), [
            'trang_thai' => 'required|string'
        ]);

        if ($validator->fails()) {
            return response()->json([
                'success' => false,
                'message' => 'Trạng thái không hợp lệ',
                'errors' => $validator->errors()
            ], 422);
        }

        $xe->trang_thai = $request->trang_thai;
        $xe->save();

        return response()->json([
            'success' => true,
            'message' => 'Thay đổi trạng thái xe thành công',
            'data' => $xe
        ], 200);
    }
}
